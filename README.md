
# EasyPermission

[![license](https://img.shields.io/github/license/blackbbc/Tucao.svg)](https://github.com/blackbbc/Tucao/blob/master/LICENSE)

EasyPermission是一个简单易用，且无多余的第三方依赖的Android6.0动态权限申请库，支持链式调用，方便快捷。

## 为什么要造这个轮子
随着Android P都开始正式发布，Android慢慢的从之前的放开权限，到现在的逐步收紧权限，目的都是为了维护Android的生态健康发展，且杜绝权限滥用的情况，同时，国内的应用市场也开始限制应用的适配和升级，必须要适配动态权限和Android8.0才能发布上应用市场，所以，急需一套权限的框架来申请和管理权限，而我查看很多知名的权限框架，并且尝试去接入的时候，发现他们的实现依赖了太多的第三方库， 本来很简单的功能，却需要依赖一堆的引用而导致包size增加，有点得不偿失的感觉，所以，轮子就这么弄出来了。

## 特点
- 链式调用，一行链式代码即可实现权限申请
- 支持自定义添加RequestPermissionRationale处理函数，可针对性的对某一个权限进行处理
- 权限申请流程可忽略，可停止
- 无第三方依赖，全部共10个java文件，copy即用，不用担心包size
- 支持单个权限、多个权限、单个权限组的权限请求
- 不依赖AppCompatSupport库
- 如果动态申请的权限没有在清单文件中注册会抛出异常
- 支持Android8.0的安装应用和悬浮窗权限

## 使用方法
```
EasyPermission.with(this)  
        .addPermissions(Permission.Group.LOCATION)      //申请定位权限组  
        .addPermissions(Permission.CALL_PHONE)          //申请打电话权限  
        .addRequestPermissionRationaleHandler(Permission.ACCESS_FINE_LOCATION, new RequestPermissionRationalListener() {  
             @Override  
             public void onRequestPermissionRational(String permission, boolean requestPermissionRationaleResult, final NextAction nextAction) {  
                //这里处理具体逻辑，如弹窗提示用户等,但是在处理完自定义逻辑后必须调用nextAction的next方法
             }  
         })  
        .addRequestPermissionRationaleHandler(Permission.CALL_PHONE, new RequestPermissionRationalListener() {  
            @Override  
             public void onRequestPermissionRational(String permission, boolean requestPermissionRationaleResult, final NextAction nextAction) {  
               //这里处理具体逻辑，如弹窗提示用户等,但是在处理完自定义逻辑后必须调用nextAction的next方法
            }  
        })   
        .request(new PermissionRequestListener() {  
            @Override  
            public void onGrant(Map<String, GrantResult> result) {  
                //权限申请返回  
            }  
  
            @Override  
            public void onCancel(String stopPermission) {  
                //在addRequestPermissionRationaleHandler的处理函数里面调用了NextAction.next(NextActionType.STOP,就会中断申请过程，直接回调到这里来  
            }  
        });
```
## addRequestPermissionRationaleHandler方法解析

这个函数，其实就是封装了系统的 **shouldShowRequestPermissionRationale** 方法，这个方法的特点是

> 1.第一次请求权限时，用户拒绝了，调用shouldShowRequestPermissionRationale()后返回true，应该显示一些为什么需要这个权限的说明
> 2.用户在第一次拒绝某个权限后，下次再次申请时，授权的dialog中将会出现“不再提醒”选项，一旦选中勾选了，那么下次申请将不会提示用户
> 3.第二次请求权限时，用户拒绝了，并选择了“不在提醒”的选项，调用shouldShowRequestPermissionRationale()后返回false
> 

而我们这个handler则采用了**链式回调**的设计，你可以设置某一个权限的handler，用来当用户拒绝过你的权限的时候，你可以视情况适当的给一些提示，提示用户给予你权限。所以，你必须在这个处理函数里面调用**nextAction**的**next**方法，来决定你对这个权限的处理，否则后续的权限流程将不会执行。**NextAction**的返回值有三个，如下：


|NextAction值|描述  |
|----|----|
|IGNORE |忽略这个权限，后面的步骤不会申请这个权限，onGrant回调也会返回IGNORE这个状态 |
|NEXT| 表示继续处理，继续下一个权限 |
|STOP|停止处理，直接回调onCancel函数 |


## request函数解析
调用这个函数的时候，就开始走权限的申请流程了，有两个回调函数，**onGrant**返回最终的权限申请结果，其中**GrantResult**(是一个**enum**类)也就是申请结果又三种状态，如下：

|GrantResult值|int映射|描述|
|----|----|----|
| IGNORE |-2|在NextAction里面传入IGNORE的时候，也会回调这个状态值，表示忽略，不申请|
| GRANT| 0|用户给予权限 |
| DENIED|-1|用户拒绝了权限 |

而**onCancel**则表示在**addRequestPermissionRationaleHandler**的处理函数里面调用了**NextAction.next(NextActionType.STOP)**,就会中断申请过程，直接回调到这里来


## 其他函数

**EasyPermission.openSettingPage(Context context);**

> 可以跳转到系统的权限设置界面，让用户更加方便的打开应用的权限

**EasyPermission.isPermissionGrant(Context context, String... permissions)**

> 判断是否已经授予了权限


License
-------

    Copyright 2018 panyiho

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
