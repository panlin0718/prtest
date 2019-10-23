## QNRTC-Android 自动化测试
   
   
### AutoTestingManager

自动测试核心类，提供相关操作接口。

#### 构造函数

```java
public AutoTestingManager()
```

#### 设置回调

```java
public void setAutoTestingCallback(AutoTestingCallback callback)
```

**参数：**

callback：回调

> 回调设置必须在初始化和开启自动化测试前。

#### 初始化

```java
public void init()
```

> 初始化成功后会触发 AutoTestingCallback#onTestingInit 回调

#### 开启自动化测试

```java
public void startAutoTesting()
```

> 开启后将会开始自动化测试，每一步都将会按顺序按时触发 AutoTestingCallback#onTestingCaseUpdated 回调

#### 停止自动化测试

```java
public void stopAutoTesting()
```

#### 上传测试结果

```java
public void uploadTestingResult(int caseId, String stepId, String stepName, boolean caseResult)
```

**参数：**

caseId: 样例ID

stepId: 步骤ID

stepName: 步骤名称

caseResult: 测试结果



### AutoTestingCallback

自动测试过程中触发的回调类。

#### 初始化成功回调

```java
void onTestingInit(TestConfig config);
```

**参数：**

config: 配置参数

#### 测试用例步骤更新回调

```java
void onTestingCaseUpdated(int caseId, int stepId, String stepName);
```

**参数：**

caseId: 用例ID

stepId: 步骤ID

stepName: 步骤名称

> 当前步骤执行时间完毕后且未触发上传结果接口将触发下一次步骤的此回调

#### 配置改变回调

```java
void onConfigChanged(TestConfig config);
```

**参数：**

config: 配置参数

#### 到达测试时长回调

```java
void onStopped();
```



### 样例代码

```java
mAutoTestingManager = new AutoTestingManager();
        mAutoTestingManager.setAutoTestingCallback(new AutoTestingCallback() {
            @Override
            public void onTestingInit(TestConfig config) {
                mRoomToken = config.getRoomToken();
                // 初始化自动化测试框架返回具体的 QNRTCSetting 参数，以此创建 QNRTCEngine
                initEngine(config);

                // 必须在自动化测试初始化后开启自动化测试！！！
                mAutoTestingManager.startAutoTesting();
            }

            @Override
            public void onTestingCaseUpdated(int caseId, int stepId, String stepName) {
                // 每一步都有一个回调，如果上一步已经执行成功（调用了上传结果的接口），会马上回调下一步
                mCaseId = caseId;
                mStepId = stepId;
                mStepName = stepName;
                switch (stepName) {
                    case "publish":
                        mEngine.publish();
                        break;
                    case "joinRoom":
                        mEngine.joinRoom(mRoomToken);
                        break;
                }
            }

            @Override
            public void onConfigChanged(TestConfig config) {
                // 当配置改变时在此重新配置
                mEngine.destroy();
                initEngine(config);
            }

            @Override
            public void onStopped() {
                finish();
            }
        });
        mAutoTestingManager.init();
```


### 参考文档

Android SDK QNRTCEngine API 文档：https://doc.qnsdk.com/rtn/android/docs/api_qnrtcengine 

Android SDK 回调文档：https://doc.qnsdk.com/rtn/android/docs/api_qnrtcengineeventlistener