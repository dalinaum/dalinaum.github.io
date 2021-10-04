---
layout: post
title:  "M1 텐서플로우 설치하기"
date:   2021-10-04 22:45:00 +0900
categories: mac
---

## 빅서에서 텐서플로우 설치하기

빅서에서 텐서플로우를 설치하기 위해서는 [텐서플로우 플러그인](https://developer.apple.com/metal/tensorflow-plugin/)을 설치해서는 안된다. 애플이 만든 스크립트로 설치를 해야한다.

### 파이썬 3.8 버전 (pyenv)

스크립트를 수행하기 전에 파이썬 버전이 3.8 버전인지 확인하자. 나는 [pyenv](https://github.com/pyenv/pyenv)를 이용했다.

```sh
brew install openssl readline sqlite3 xz zlib
brew install pyenv
```

`pyenv`가 설치가 되면 환경 설정을 해야 한다.

```sh
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zprofile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zprofile
echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

만약 `eval` 과정에서 에러가 생긴다면 경로를 적절히 바꾸어야 한다. 나는 `eval "$(pyenv init --path)"`를 `eval "$(/usr/local/m1-homebrew/bin/pyenv init --path)"`로 바꾸었다.

### 텐서플로우 macOS 설치

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/apple/tensorflow_macos/master/scripts/download_and_install.sh)"
```

나는 파이썬 버전이 맞지 않아 다음의 명령어로 설치를 마무리했다. `/var/folders/zz/lgvb77nx6d53ppgc08_yp7v00000gn/T/tmp.RN5VHiRicK/` 등의 임시폴더는 각자 다르기 떄문에 에러 메시지를 확인하자.

```sh
/var/folders/zz/lgvb77nx6d53ppgc08_yp7v00000gn/T/tmp.RN5VHiRicK/tensorflow_macos/install_venv.sh --prompt --python /Users/dalinaum/.pyenv/shims/python
```

`--python` 뒤에 `pyenv`가 설치한 파이썬 경로를 지정했다. `/Users/dalinaum/.pyenv/shims/python` 이 경로도 사용자마다 다를 것이다.

설치가 완료되면 텐서플로우 환경을 사용해보자.

```sh
source ~/tensorflow_macos_venv/bin/activate
```

### mnist 테스트

제대로 설치가 되었는지는 mnist 예제로 해보았다.

```py
from tensorflow.keras.datasets import mnist
from tensorflow.keras.layers import Flatten, Dense, Dropout
from tensorflow.keras.models import Sequential

(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = Sequential([
    Flatten(input_shape=(28, 28)),
    Dense(128, activation='relu'),
    Dropout(0.2),
    Dense(10, activation='softmax')
])

model.compile(optimizer='sgd',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=3)
```

## 몬트레이 버전 텐서플로우

[애플이 공식적으로 제안하는 방식](https://developer.apple.com/metal/tensorflow-plugin/)인데 문제는 몬트레이 버전 (macOS 12.0+) 이상에서만 사용할 수 있다. 아직은 사용하지 말자.

### Miniforge 설치

첫 단계는 [Miniforge](https://github.com/conda-forge/miniforge)를 다운 받아 설치하는 것이다.

```sh
curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
zsh Miniforge3-MacOSX-arm64.sh
```

설치 직후에는 쉘에서 다음 커맨드를 수행하자. 이후는 수행할 필요가 없다.

```sh
source ~/miniforge3/bin/activate
```

나는 굳이 `base`버전을 쓰고 싶지 않기 때문에 아래의 커맨드를 입력한다.

```sh
conda config --set auto_activate_base false
```

### 텐서플로우 의존성 설치하기

```sh
conda create --name tf-test python=3.8
conda activate tf-test
```

```sh
conda install -c apple tensorflow-deps
```

```sh
pip install tensorflow-macos tensorflow-metal
```

빅서에서 수행하면 `adam` 옵티마이저가 제대로 실행되지 않는다. 절대 빅서에서 수행하지 말자.