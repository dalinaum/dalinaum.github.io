---
layout: post
title:  "Glide의 이미지 뷰에서 비트맵 얻기"
date:   2015-07-07 00:42:00 +0900
categories: android 
---

[Glide](https://github.com/bumptech/glide)는 좋은 라이브러리이지만 이미지 뷰에서 이미지를 뽑는 것이 쉽지 않습니다. 이미지 뷰가 `DrawableBitmap`을 가지지 않고 다양한 `Drawable`을 가지기 때문입니다.

이미지가 캐쉬에 있는 경우에는 `GlideBitmapDrawable`을 가질 수 있습니다. 그렇지 않은 경우에는 `TransitionDrawable`을 가질 수도 있는데 이 경우에는 레이어에 `GlideBitmapDrawable`을 직접 가지는 경우와 `SquaringDrawable`을 가지는 경우를 보았습니다.

이 경우들을 상정하고 작성한 코드는 아래와 같습니다. `instanceOfDrawable`가 도배되어 좀 더러운 코드입니다.

```
Bitmap bitmap = null;
Drawable drawable = imageCard.getImageView().getDrawable();
if (drawable instanceof GlideBitmapDrawable) {
    bitmap = ((GlideBitmapDrawable) drawable).getBitmap();
} else if (drawable instanceof TransitionDrawable) {
    TransitionDrawable transitionDrawable = (TransitionDrawable) drawable;
    int length = transitionDrawable.getNumberOfLayers();
    for (int i = 0; i < length; ++i) {
        Drawable child = transitionDrawable.getDrawable(i);
        if (child instanceof GlideBitmapDrawable) {
            bitmap = ((GlideBitmapDrawable) child).getBitmap();
            break;
        } else if (child instanceof SquaringDrawable) {
            if (child.getCurrent() instanceof GlideBitmapDrawable) {
                bitmap = ((GlideBitmapDrawable) child.getCurrent()).getBitmap();
                break;
            }
        }
    }
} else if (drawable instanceof SquaringDrawable) {
    bitmap = ((GlideBitmapDrawable) drawable.getCurrent()).getBitmap();
}
```

Glide가 생성하는 모든 `Drawable`을 고려한 것이 아니기 때문에 빠져있는 케이스가 존재할지도 모르겠는데 제가 쓰는 케이스에서는 이 정도면 대부분은 커버하는 것 같습니다.

이왕 커스텀 `Drawable`을 쓴다면 `Bitmap`을 좀 더 손쉽게 추출할 수 있는 방법을 제공해줬으면 더 좋았을 것 같은데 조금 아쉽네요. 
