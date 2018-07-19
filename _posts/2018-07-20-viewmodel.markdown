---
layout: post
title:  "안드로이드 ViewModel"
date:   2018-07-20 01:47:00 +0900
categories: android
---

안드로이드 뷰모델 [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)은 비교적 새로운 세상이며 오해가 많은 영역 중 하나다. 이 글에서는 사용자가 이미 `ViewModel`의 간략한 사용법을 알고 있다 가정하며 더 깊은 이해를 위해 구현부를 살펴보도록 한다.

`ViewModel` 객체인 `UserViewModel`이 있다면 이를 생성하기 위해 다음과 같이 사용한다. 이 호출을 한 단계씩 따라가보자.

`ViewModel`의 사용법이 생소하다면 먼저 사용법을 설명한 글들을 읽고 오자.

```
val viewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)
```

`ViewModelProviders.of()` 메서드를 통해 `ViewModelProvider` 객체를 얻고 `ViewModelProvider.get()` 메서드를 통해 `ViewModel` 인스턴스를 얻는 것이다.

`ViewModelProviders.of()`을 따라 가자.

```
public static ViewModelProvider of(@NonNull FragmentActivity activity) {
    return of(activity, null);
}
```

시그내쳐만 다른 동명의 메서드로 호출한다.

```
public static ViewModelProvider of(@NonNull FragmentActivity activity,
        @Nullable Factory factory) {
    Application application = checkApplication(activity);
    if (factory == null) {
        factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
    }
    return new ViewModelProvider(ViewModelStores.of(activity), factory);
}
```

`checkApplication`은 액티비티를 통해 애플리케이션을 가져온다.

```
private static Application checkApplication(Activity activity) {
    Application application = activity.getApplication();
    if (application == null) {
        throw new IllegalStateException("Your activity/fragment is not yet attached to "
                + "Application. You can't request ViewModel before onCreate call.");
    }
    return application;
}
```

애플리케이션을 가져온 후 `ViewModelProvider.AndroidViewModelFactory.getInstance(application)`를 이용하여 팩토리를 만든다.

```
public static AndroidViewModelFactory getInstance(@NonNull Application application) {
    if (sInstance == null) {
        sInstance = new AndroidViewModelFactory(application);
    }
    return sInstance;
}
```

팩토리는 흔히 효율을 위해 싱글톤을 이용해 만들어진다. `sInstance`가 그 증거다.

팩토리는 `create` 메서드를 가지고 있다.

```
public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
    if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
        //noinspection TryWithIdenticalCatches
        try {
            return modelClass.getConstructor(Application.class).newInstance(mApplication);
        } catch (NoSuchMethodException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        } catch (InstantiationException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        } catch (InvocationTargetException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        }
    }
    return super.create(modelClass);
}
```

`create` 메서드의 핵심적인 부분은 객체를 생성하는 부분이다.

```
return modelClass.getConstructor(Application.class).newInstance(mApplication);
```

클래스의 생성자를 가져와서 인스턴스를 생성하는 코드다. 기본 팩토리는 사용자가 원하는 뷰 모델 객체를 단순히 생성해서 전달한다.

다시 `ViewModelProviders.of()`로 돌아가자.

```
public static ViewModelProvider of(@NonNull FragmentActivity activity,
        @Nullable Factory factory) {
    ...
    return new ViewModelProvider(ViewModelStores.of(activity), factory);
}
```

`ViewModelProviders.of()`은 `ViewModelProvider`를 새롭게 만든다. `ViewModelProvider`를 재사용하지 않는 부분에서 생성자에 들어가는 파라미터 두개 `ViewModelStores.of(activity)`와 `factory`가 재사용할 수 있을 것을 의심해봐야한다.

`factory`는 싱글턴으로 효율적으로 가져올 수 있다는 것을 확인했기 때문에 남은 것은 `ViewModelStores.of(activity)`가 재사용할 수 있는지 확인하자.

```
public static ViewModelStore of(@NonNull FragmentActivity activity) {
    if (activity instanceof ViewModelStoreOwner) {
        return ((ViewModelStoreOwner) activity).getViewModelStore();
    }
    return holderFragmentFor(activity).getViewModelStore();
}
```

두가지 경로가 있다.

1. 액티비티가 `ViewModelStoreOwner`이라 `ViewModelStoreOwner.getViewModelStore()`로 가져올 수 있는 상황.
2. `HolderFragment.getViewModelStore()`를 호출하기 위해 `holderFragmentFor()`를 사용하기.

1번의 경우는 예외적인 경우다. `ViewModelStoreOwner`은 인터페이스이고 우리가 만든 `Activity`가 그것을 구현하지 않았다고 볼 수 있다.

그럼 `holderFragmentFor(activity)`가 호출되었다고 볼 수 있다.

```
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public static HolderFragment holderFragmentFor(FragmentActivity activity) {
    return sHolderFragmentManager.holderFragmentFor(activity);
}
```

`HolderFragmentManager.holderFragmentFor()`로 포워딩된다.

```
HolderFragment holderFragmentFor(FragmentActivity activity) {
    FragmentManager fm = activity.getSupportFragmentManager();
    HolderFragment holder = findHolderFragment(fm);
    if (holder != null) {
        return holder;
    }
    holder = mNotCommittedActivityHolders.get(activity);
    if (holder != null) {
        return holder;
    }

    if (!mActivityCallbacksIsAdded) {
        mActivityCallbacksIsAdded = true;
        activity.getApplication().registerActivityLifecycleCallbacks(mActivityCallbacks);
    }
    holder = createHolderFragment(fm);
    mNotCommittedActivityHolders.put(activity, holder);
    return holder;
}
```

코드를 간략화해보면 다음과 같다.

1. `findHolderFragment`를 이용해서 이미 그 액티비티에 `HolderFragment`가 있는지 확인한다.
2. `mNotCommittedActivityHolders`에 남아있는 `HolderFragment`가 있는지 확인한다.
3. `HolderFragment`를 만들고 `mNotCommittedActivityHolders`에 넣어둔다.

`HolderFragment`를 생성하는 부분은 다음과 같다.

```
private static HolderFragment createHolderFragment(FragmentManager fragmentManager) {
    HolderFragment holder = new HolderFragment();
    fragmentManager.beginTransaction().add(holder, HOLDER_TAG).commitAllowingStateLoss();
    return holder;
}
```

`HolderFragment`가 핵심적인 부분으로 보인다. 대체 왜 `HolderFragment`까지 내려왔을까?

정답을 알기 위해 `HolderFragment`의 일부를 살펴보자.

```
public class HolderFragment extends Fragment implements ViewModelStoreOwner {
    ...
    private ViewModelStore mViewModelStore = new ViewModelStore();

    public HolderFragment() {
        setRetainInstance(true);
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        sHolderFragmentManager.holderFragmentCreated(this);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        mViewModelStore.clear();
    }

    @NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        return mViewModelStore;
    }
}
```

생성자에 무언가 보인다.

```
public HolderFragment() {
    setRetainInstance(true);
}
```

이 프래그먼트가 `setRetainInstance(true)`로 선언되어 있다. 따라서 `HolderFragment`는 단말기의 로테이션 회전 등의 설정 변경에 살아남게 된다. `ViewModel`이 왜 화면 전환 등에 버틸 수 있는지 단서가 보인다.

그 다음으로 `HolderFragment`가 `ViewModel`들을 가지고 있는 것을 확인 할 수 있다.

```
private ViewModelStore mViewModelStore = new ViewModelStore();
```

`ViewModelStore`가 해쉬맵으로 `ViewModel`등을 가지고 있다.

```
public class ViewModelStore {
    ...
    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }
    ...
}
```

`HolderFragment`가 가진 `onCreate`과 `onDestroy`를 통해 생명주기도 관리한다.

```
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    sHolderFragmentManager.holderFragmentCreated(this);
}

@Override
public void onDestroy() {
    super.onDestroy();
    mViewModelStore.clear();
}
```

생성될 때 (정확히는 `onCreate` 상황에서) `HolderFragmentManager.holderFragmentCreated()`가 호출되고 해제될 때 (정확히는 `onDestroy` 때) `ViewModelStore.clear()`를 호출해서 뷰모델을 정리한다.

`HolderFragment`가 생성될 때 `HolderFragmentManager.holderFragmentCreated()`를 왜 호출했을까?

```
void holderFragmentCreated(Fragment holderFragment) {
    Fragment parentFragment = holderFragment.getParentFragment();
    if (parentFragment != null) {
        mNotCommittedFragmentHolders.remove(parentFragment);
        parentFragment.getFragmentManager().unregisterFragmentLifecycleCallbacks(
                mParentDestroyedCallback);
    } else {
        mNotCommittedActivityHolders.remove(holderFragment.getActivity());
    }
}
```

아까 봤던 `mNotCommittedActivityHolders`가 여기에서 관리된다. 결국 `mNotCommittedActivityHolders`는 아직 `onCreate`가 안된 `HolderFragment`가 있는지 체크하기 위한 곳인 셈이다.

처음보는 `mNotCommittedFragmentHolders`등도 있지만 이들은 프래그먼트에서 `ViewModel`을 사용할 떄를 위한 코드다. 지금은 액티비티에서 뷰모델을 사용하고 있고 프래그먼트 케이스의 경우에는 직접 따라가보자.

그럼 이제 `HolderFragment`를 생성하거나 가져오는 부분으로 돌아가 정보를 업데이트 하자.

```
HolderFragment holderFragmentFor(FragmentActivity activity) {
    FragmentManager fm = activity.getSupportFragmentManager();
    HolderFragment holder = findHolderFragment(fm);
    if (holder != null) {
        return holder;
    }
    holder = mNotCommittedActivityHolders.get(activity);
    if (holder != null) {
        return holder;
    }

    if (!mActivityCallbacksIsAdded) {
        mActivityCallbacksIsAdded = true;
        activity.getApplication().registerActivityLifecycleCallbacks(mActivityCallbacks);
    }
    holder = createHolderFragment(fm);
    mNotCommittedActivityHolders.put(activity, holder);
    return holder;
}
```

1. `findHolderFragment`를 이용해서 이미 그 액티비티에 `HolderFragment`가 있는지 확인한다.
2. ~~ `mNotCommittedActivityHolders`에 남아있는 `HolderFragment`가 있는지 확인한다. ~~ `mNotCommittedActivityHolders`를 확인하여 아직 `onCreate`가 호출되지 않은 `HolderFragment`는 재사용한다.
3. `HolderFragment`를 만들고 `mNotCommittedActivityHolders`에 넣어둔다.

다시 위로 올라가 `ViewModelProvider.of()`로 가자.

```
public static ViewModelProvider of(@NonNull FragmentActivity activity,
        @Nullable Factory factory) {
    ...
    return new ViewModelProvider(ViewModelStores.of(activity), factory);
}
```

`ViewModelStores.of(activity)`가 반환한 `ViewModelStore`는 결국 `HolderFragment`가 수명이 있는 동안 유지되기 때문에 효율적으로 가져올 수 있다. `factory`는 당연히 싱글턴이고.

한 단계 더 올라가자.

```
val viewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)
```

`ViewProvider.get()` 메서드가 호출되었다. 이 메서드의 모양은 아래와 같다.

```
public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
    String canonicalName = modelClass.getCanonicalName();
    if (canonicalName == null) {
        throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
    }
    return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
}


public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
    ViewModel viewModel = mViewModelStore.get(key);

    if (modelClass.isInstance(viewModel)) {
        //noinspection unchecked
        return (T) viewModel;
    } else {
        //noinspection StatementWithEmptyBody
        if (viewModel != null) {
            // TODO: log a warning.
        }
    }

    viewModel = mFactory.create(modelClass);
    mViewModelStore.put(key, viewModel);
    //noinspection unchecked
    return (T) viewModel;
}
```

적당히 키의 이름을 정의해서 `ViewModelStore`로 부터 가져오는 것을 볼 수 있다.

1. 개별 액티비티는 `HolderFragment` 프래그먼트를 이용해서 `ViewModel`를 생성하거나 가져온다.
2. 우리가 만든 `ViewModel`은 `ViewModelStore`에 키-밸류 형태로 저장된다.
3. `ViewModelStore`는 상태 변환에 안전한 프래그먼트 `HolderFragment`에 보관된다.
