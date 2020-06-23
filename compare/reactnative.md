#### react native相关代码展示

>模拟器页面展示效果为丰密app, 时间关系实现度大约70%, 包括首页,流程,数据,列表,详情

<!-- ![丰密APP演示](../gif/index.gif) -->

#### 用到的技术和实现方案等
1. `react native`
2. `react navigation`(包括`tab`, `stack`路由)
3. 将`fetch`作为请求库
4. 自定义组件封装, 比如`loading`, `toast`, 搜索框
5. `session`用`sso-sit2`获取到, 数据是`cnsapp-sit2`

#### 遇到的一些坑和难点
1. 搭建环境会比较痛苦, 搭建好了之后就容易多了
2. `install`一个外部插件之后, 环境经常崩, 不过大多数情况下都可以`google`
3. 需要经常去官网查询相关知识点和使用文档, 前期开发比较费时, 等到大多数方法,`api`了然于胸的时候开发就快了.

#### 开发感受到的不一样
1. `react native`提供了很多方法或者组件, 比如长列表`FlatList`以及相关的下拉加载, 滚动加载等都支持, 非常方便, 性能也非常高.
2. 支持`flex`布局, `css`属性都是驼峰写法, 和常规的`flex`差别不大, 稍微注意即可


```javascript
// app.js 代码有删减

import 'react-native-gesture-handler';
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createStackNavigator } from '@react-navigation/stack';

// 公共组件
import Toast from './views/common/toast';

// 页面相关
import Home from './views/page/home';
import Flow from './views/page/flow';
import Data from './views/page/data';
import BottomIcon from './views/common/bottomIcon';

// 子页面
import List from './views/page/flow/list';
import Detail from './views/page/detail/detail';

// 导航相关
const Tab = createBottomTabNavigator();
const SettingsStack = createStackNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={({ route }) => ({
          tabBarIcon: ({ focused, color, size }) => {
            let iconName;
            if (route.name === 'Home') {
              iconName = focused ? 'home-selected' : 'home';
            } else if (route.name === 'Flow') {
              iconName = focused ? 'flow-selected' : 'flow';
            } else{
              iconName = focused ? 'data-selected' : 'data';
            }
            return <BottomIcon iconName={iconName}/>;
          },
        })}
      >
        <Tab.Screen name="Home" component={Home}>
        </Tab.Screen>
        <Tab.Screen name="Flow">
          {
            () => (
              <SettingsStack.Navigator headerMode="none" initialRouteName="Flow">
                <SettingsStack.Screen name="Flow" component={Flow} />
                <SettingsStack.Screen name="List" component={List} />
                <SettingsStack.Screen name="Detail" component={Detail} />
              </SettingsStack.Navigator>
            )
          }
        </Tab.Screen>
        <Tab.Screen name="Data" component={Data} />
      </Tab.Navigator>
      <Toast ref={toast => global.$toast = toast}/>
    </NavigationContainer>
  );
};

export default App;
```

```js
// home.js  代码有删减

import React, { Component } from 'react'
import {
  View,
  Text,
  StyleSheet,
  DeviceEventEmitter,
  ImageBackground,
  Button
} from 'react-native';
import Header from '../common/header';
import Nav from '../components/nav';
import MyTasks from '../components/myTasks';
import Loading from '../common/loading';

import http from '../api/index';

export default class Home extends Component {
  constructor(props){
    super(props);
    
    this.state = {
      addNum: 0,
      existNum: 0,
    }
  }

  // 通过sso获取code
  getCode = async () => {
    let url = 'https://sso-sit2.fcbox.com/mobileAuth/newMLogin';
    const params = {
      userName: '002158',
      password: 'D55Jc7/ 0waJEQoy26m7Z8l7lZOmJDROWWDN6FWXTZb6o38olXj8RQvpPydQRWJl1L/a/H9rkTAOT8CzfT / 2i4c4V2L6bmuqvN6sx + Uvjx8veKJlVD2fZhZ3eMFygaeqzVEp / +b5vIMJ5SCHqi9DHGj7Uuhg04WRq7EJn1VcttQc=',
      clientId: 'fm',
      rememberMe: true,
    }
    return http.postForm(url, params)
  };

  // 通过code来获取SESSION
  getSession = async (code) => {
    let url = `loginController/getSeesionId?code=${code}`;
    return http.get(url)
  };

  getMaliceOccupyNum = async () =>{
    let url = `appApi/maliceOccupy/getMaliceOccupyNum`;
    http.get(url).then(res=>{
      console.log('getMaliceOccupyNum', res)
      if (res.code == 0 && res.data) {
        this.setState({
          addNum: res.data
        })
      }
    }).catch(err=>{
      console.log('err', err)
    })
  }

  getSiteHomeData = async () => {
    let url = `appApi/siteHomeData`;
    http.post(url).then(res => {
      console.log('getSiteHomeData', res)
      if (res.code == 0 && res.data) {
        this.setState({
          existNum: res.data.saveSiteCnt
        })
      }
    }).catch(err => {
      console.log('err', err)
    })
  }

  async componentDidMount(){
    DeviceEventEmitter.emit('loading', true);
    let code = '';
    let SESSION = '';
    try {
      const res = await this.getCode();
      if(res.code == 0 && res.data){
        code = res.data
      }
    } catch (error) {
      code = null
      console.log('error', error)
    }
    try {
      if (code){
        const res = await this.getSession(code);
        if (res.code == 0 && res.data) {
          SESSION = res.data
        }
        DeviceEventEmitter.emit('loading', false);
      }
    } catch (error) {
      SESSION = null
      console.log('error', error)
    };
    DeviceEventEmitter.emit('loading', false);
    this.getMaliceOccupyNum();
    this.getSiteHomeData();
  }

  render() {
    const { addNum, existNum } = this.state
    let bg = require('../imgs/bg.png')
    return (
        <View style={styles.container}>
          <Header />
          <View style={styles.box}>
            <ImageBackground source={bg}>
              <Text style={styles.bigTitle}>丰收系统</Text>
            </ImageBackground>
          </View>
          <View style={styles.content}>
            <Nav />
            <MyTasks addNum={addNum} existNum={existNum}/>
          </View>
          <Loading />
        </View>
    )
  }
}

const styles = StyleSheet.create({
  box: {
    height: 200
  },
  bigTitle: {
    height: 32,
    fontSize: 24,
    paddingLeft: 15,
    fontFamily: 'PingFangSC - Medium, PingFang SC',
    fontWeight: 'bold',
    color: 'rgba(255, 255, 255, 1)',
    lineHeight: 33,
  },
  content:{
    position: 'absolute',
    top:80,
  },
})
```