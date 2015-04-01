---
layout: post
title:  "안녕 React Native"
date:   2015-03-27 16:33:06 +0900
categories: react ios
---
페이스북이 드디어 [React Native][react-native]를 발표했다. 자바스크립트 플랫폼 [React][react]에서 네이티브 애플리케이션을 개발할 수 있게 하는 기술이다. 안드로이드 환경에 대해서 얘기했던 키노트와는 달리 공개된 코드는 iOS 전용이고 Mac 환경만 출시되었다. 그렇기 때문에 React Native를 사용하기 위해 맥과 [Xcode][xcode]와 [Homebrew][homebrew]가 필요하다.

환경이 준비되었으면 이제 `react-native-cli`를 설치하자.

````
brew install node
brew install watchman
brew install flow
npm install -g react-native-cli

````

환경이 준비되었으면 첫번째 프로젝트를 만들어보자.

````
react-native init HelloWorld
````

설치가 완료되면 아래와 같은 화면이 출력된다.

````
Setting up new React Native app in /Users/lameduck/project/HelloWorld

Next steps:

   Open /Users/lameduck/project/HelloWorld/HelloWorld.xcodeproj in Xcode
      Hit Run button
````

굳이 저렇게 할 필요없이 컨맨드라인에서 아래와 같이 입력하자.

````
cd HelloWorld/
open HelloWorld.xcodeproj/
````

![Xcode 이미지]({{ site.url }}/assets/react-xcode.png)

어 자바스크립트 파일이 없네? 하지만 쉘에서 파일 목록을 보면 자바스크립트 파일이 있다.

````
ls
HelloWorld.xcodeproj    iOS         index.ios.js        node_modules        package.json
````

index.ios.js라니 웹 스럽기도 하고 아이폰 스럽기도 한 파일명이다. index.ios.js파일을 살펴보자.

{% highlight javascript %}

/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
} = React;

var HelloWorld = React.createClass({
  render: function() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit index.ios.js{'\n'}
          Press Cmd+R to reload
        </Text>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
  },
});

AppRegistry.registerComponent('HelloWorld', () => HelloWorld);
{% endhighlight %}

앱을 수행시켜보자. Xcode에서 앱을 수행하면 된다.

![React Native 수행 화면]({{ site.url }}/assets/react-iphone.png)

문구를 `Welcome to React Native!`에서 `Hello React Native!'로 바꿔보자. `HelloWorld` 콤퍼넌트를 찾아 수정해야 한다.

{% highlight javascript %}
var HelloWorld = React.createClass({
  render: function() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit index.ios.js{'\n'}
          Press Cmd+R to reload
        </Text>
      </View>
    );
  }
});
{% endhighlight %}

여기에서 `Welcome to React Native!`를 `Hello React Native!`로 바꾸자. 바꾼 후에 에물레이터에서 `Cmd+R`를 눌러 앱을 업데이트하면 다시 컴파일 할 필요없이 앱이 갱신된다.

[react]:        http://facebook.github.io/react/
[xcode]:        https://developer.apple.com/xcode/downloads/
[homebrew]:     http://brew.sh/
[react-native]: http://facebook.github.io/react-native/

