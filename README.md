# IM SDK for iOS 使用指南

## IM SDK 常见问题->[IM FAQ详解](https://github.com/youmesdk/wiki/blob/master/IMFAQ.md)

## 导入IM SDK

- 将`im_ios`复制到工程的根目录下。
- 在`Build Settings -> Search Paths -> Header Search Paths`中添加头文件路径：`projectRoot/im_ios/include/`
- 在`Build Settings -> Search Paths -> Framework Search Paths`中添加框架文件路径：`projectRoot/im_ios/lib/ios/`
- 在`Build Settings -> Search Paths -> Library Search Paths`中添加库文件路径：`projectRoot/im_ios/lib/ios/`
- 在`Build phases ->Complle Sources 中将sdk中include文件夹里面的所有.m  .mm文件添加到这里来`
- 在`Build Phases  -> Link Binary With Libraries`添加如下依赖库：
 >`libc++.1.tbd`  
 >`libsqlite3.0.tbd`  
 >`libresolv.9.tbd`  
 >`SystemConfiguration.framework`  
 >`libYouMeCommon.a`  
 >`libyim.a`  
 >`libz.tbd`  
 >`CoreTelephony.framework`  
 
 >`AVFoundation.framework`  
 >`AudioToolbox.framework`   
 >`CoreLocation.framework`  
   
 >`iflyMSC.framework`       //如果是带语音转文字的sdk需要添加。  
 
 >`AliyunNlsSdk.framework`  //如果是带语音转文字的sdk需要添加，该库是动态库，需要在`General -> Embedded Binaries`也进行添加。  
 
 >`USCModule.framework`     //如果是带语音转文字的sdk需要添加。  
 


- **需要为iOS10以上版本添加录音权限配置**
iOS 10 使用录音权限，需要在`info`新加`Privacy - Microphone Usage Description`键，值为字符串，比如“语音聊天需要录音权限”。
首次录音时会向用户申请权限。配置方式如图(选择Private-Microphone Usage Description)。
![iOS10录音权限配置](https://www.youme.im/doc/images/im_iOS_record_config.jpg)

- **LBS添加获取地理位置权限**
若使用LBS，需要在`info`新加`Privacy - Location Usage Description`键，值为字符串，比如“查看附近的玩家需要获取地理位置权限”。首次使用定位时会向用户申请权限，配置方式如上图录音权限。

## 初始化
### 引入SDK头文件

``` obj-c
#import "YIMClient.h"

```

### 初始化SDK 

- 接口：

  ``` obj-c
  -(YIMErrorcodeOC)InitWithAppKey:(NSString *)strAppKey appSecurityKey:(NSString*)strAppSecurity serverZone:(YouMeIMServerZoneOC)serverZone;

  [[YIMClient GetInstance] InitWithAppKey:@"appkey" appSecurityKey:@"secretkey" serverZone:YouMeIMServerZone_China];
  ```
  
- 参数说明：
  `appkey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  `appSecurityKey`：用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看  
  `serverZone`：设置IM服务器区域，详细定义参见[服务器部署地区定义](#服务器部署地区定义) 
  
- 返回值：
  `YIMErrorcodeOC` ：错误码，详细描述见[错误码定义](#错误码定义)

- 回调：
  无。

### 设置监听 
回调代理类需实现 protocol `<YoumeIMCallbackProtocol>`

- 接口：

  ``` obj-c
  -(void)SetDelegate:(id<YIMCallbackProtocol>) outDelegate;
  ```

### 重连回调

- 开始重连回调：

``` obj-c
- (void) OnStartReconnect {}
```

- 重连结果回调：

``` obj-c
- (void) OnRecvReconnectResult:(ReconnectResultOC)result {}
```

- 参数说明：
  `result`：重连结果，枚举类型，
           YIMReconnectResult_Success=0 //重连成功
           YIMReconnectResult_Fail_Again=11 //重连失败，再次重连
           YIMReconnectResult_Fail=2 //重连失败 
    
### 录音音量回调
音量值范围：0~1，频率：1s 约2次

- 接口：

``` obj-c
- (void) OnRecordVolumeChange:(float) volume {}
```

- 参数说明：
  `volume`：音量值

## 用户管理
### 用户登录

- 接口：

  ```obj-c
  -(void) Login:(NSString *)userName password:(NSString *)password token:(NSString*) token callback:(loginCBType) callback;

  [[YIMClient GetInstance] Login:@"userid" password:@"123456" token:@"" callback:^(YIMErrorcodeOC errorcode, NSString *userID) {        
        if(errorcode == YouMeIMCode_Success)
        {
           NSLog(@"userID:%@ 登录成功",userID);
        }
        else
        {
           NSLog(@"userID:%@ 登录失败，err: ",userID,errorcode);
        }
   }];
  ```

- 参数说明：
  `userid`：用户id，由调用者分配，不可为空字符串，只可由字母或数字或下划线组成，长度限制为255字节
  `password`：用户密码，不可为空字符串，一般固定为"123456"即可
  `token`：使用服务器token验证模式时使用该参数，否则使用空字符串，由restAPI获取token值
  `callback`： 登录回调

- 备注：
  `loginCBType`类型定义如下：

  ``` obj-c
  typedef void(^loginCBType)(YIMErrorcodeOC errorcode, NSString* userID);
  ```

- 参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `userID`：用户ID

### 用户登出
退出当前登录的账号

- 接口：

``` obj-c
-(void) Logout:(logoutCBType)callback;

[[YIMClient GetInstance] Logout:^(YIMErrorcodeOC errorcode) {
        if(errorcode == YouMeIMCode_Success)
        {
           NSLog(@"登出成功");
        }
        else
        {
           NSLog(@"登出失败，err: ",errorcode);
        }       
    }];
```

- 参数说明：  
  `callback`： 登出回调

- 备注：
  `logoutCBType`类型定义如下：

  ``` obj-c
  typedef void(^logoutCBType)(YIMErrorcodeOC errorcode);
  ```

- 参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)  

### 被用户踢出通知
同一个用户ID在多台设备上登录时，后登录的会把先登录的踢下线，收到OnKickOff通知。

- 接口

``` obj-c
- (void) OnKickOff {}
```

### 用户信息
#### 设置用户的详细信息
- 接口：

``` obj-c
-(YIMErrorcodeOC) SetUserInfo:(NSString*) userInfo;
```

- 参数：
  `userInfo`： 格式为json {"nickname":"","server_area_id":"","server_area":"","location_id":"","location":"","level":"","vip_level":""} (可以添加其他字段)

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义) 

#### 获取指定用户的详细信息
- 接口：

``` obj-c
-(void) GetUserInfo:( NSString * )userID callback:(getUserInfoCBType)callback;
```

- 参数：
  `userID`：要查询的用户ID
  `callback`： 查询用户详细信息回调

- 备注：
  `getUserInfoCBType`类型定义如下：

  ``` obj-c
  typedef void(^getUserInfoCBType)(YIMErrorcodeOC errorcode, NSString* userID, NSString* userInfo);
  ```
  
- 参数：
  `errorcode`：错误码
  `userID`：用户ID
  `userInfo`：用户详细信息，格式为json ，{"nickname":"","server_area_id":"","server_area":"","location_id":"","location":"","level":"","vip_level":""} (可以添加其他字段)

#### 查询用户在线状态
- 接口：

``` obj-c
-(void) QueryUserStatus:( NSString * )userID callback:(queryUserStatusCBType)callback;
```

- 参数：
  `userID`：要查询的用户ID
  `callback`： 查询用户在线状态回调

- 备注：
  `queryUserStatusCBType`类型定义如下：

  ``` obj-c
  typedef void(^queryUserStatusCBType)(YIMErrorcodeOC errorcode, NSString* userID, YIMUserStatusOC status);
  ```
  
- 参数：
  `errorcode`：错误码，查询请求是否成功的通知 
  `userID`：查询的用户ID  
  `status`：登录状态，枚举类型，
            YIMSTATUS_ONLINE=0  //在线
            YIMSTATUS_OFFLINE=1 //离线

## 频道管理
### 加入频道

- 接口：

  ``` obj-c
  -(void) JoinChatRoom:(NSString *)roomID callback:(joinRoomCBType)callback;

  [[YIMClient GetInstance] JoinChatRoom:@"room_w123" callback:^(YIMErrorcodeOC errorcode, NSString *roomID) {
                         
         if (errorcode == YouMeIMCode_Success)
         {
            NSLog(@"加入频道:%@ 成功",roomID);
         }
         else
         {
            NSLog(@"加入频道:%@ 失败，err:%d",roomID,errorcode);
         }
  }];
  ```

- 参数说明：
  `roomID`： 请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节
  `callback`： 加入频道回调

- 备注：
  `joinRoomCBType`类型定义如下：

  ``` obj-c
  typedef void(^joinRoomCBType)(YIMErrorcodeOC errorcode, NSString* roomID);
  ```

- 参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `roomID`：频道ID

### 离开频道

- 接口：

  ``` obj-c
  -(void) LeaveChatRoom:(NSString *)roomID callback:(leaveRoomCBType) callback;

  [[YIMClient GetInstance] LeaveChatRoom:@"room_w123" callback:^(YIMErrorcodeOC errorcode, NSString *roomID) {
       if (errorcode == YouMeIMCode_Success)
       {
            NSLog(@"离开频道:%@ 成功",roomID);
       }
       else
       {
            NSLog(@"离开频道:%@ 失败，err:%d",roomID,errorcode);
       }
  }];
  ```

- 参数说明：
  `roomID`： 频道ID
  `callback`： 离开频道回调
 
- 备注：
  `leaveRoomCBType`类型定义如下：

  ``` obj-c
  typedef void(^leaveRoomCBType)(YIMErrorcodeOC errorcode,NSString* roomID);
  ```  

- 参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `roomID`：频道ID  

### 离开所有频道

- 接口：

  ``` obj-c
  -(void) LeaveAllChatRooms:(leaveAllRoomCBType)callback;

  [[YIMClient GetInstance] LeaveAllChatRooms:^(YIMErrorcodeOC errorcode) {
       
        if(errorcode == YouMeIMCode_Success)
        {
           NSLog(@"离开所有房间成功");
        }
        else
        {
           NSLog(@"离开所有房间失败，err: ",errorcode);
        }  
  }];
  ```

- 备注：
  `leaveAllRoomCBType`类型定义如下：

  ``` obj-c
  typedef void(^leaveAllRoomCBType)(YIMErrorcodeOC errorcode);
  ```  

- 参数：
  `errorcode`：错误码

### 用户进出频道通知
此功能默认不开启，需要的请联系我们开启此服务。联系我们，可以通过专属游密支持群或者技术支持的大群。

小频道(小于100人)内，当其他用户进入频道，会收到`OnUserJoinRoom`通知

- 原型：

``` obj-c
- (void) OnUserJoinRoom:(NSString*)roomId userID:(NSString*)userId {}
```

- 参数：
  `roomId`：频道ID
  `userID`：用户ID

小频道(小于100人)内，当其他用户退出频道，会收到`OnUserLeaveRoom`通知

- 原型：

``` obj-c
- (void) OnUserLeaveRoom:(NSString*)roomId userID:(NSString*)userId {}
```

- 参数：
  `roomId`：频道ID
  `userID`：用户ID

### 获取频道成员数量
获取进入频道的用户数量。

- 原型：

  ``` obj-c
  -(void) GetRoomMemberCount:(NSString *)roomID callback:(getRoomMemberCountCBType) callback;
  ```
  
- 参数：
  `roomID `：频道ID(已成功加入此频道才能获取该频道的人数)
  `callback`： 获取频道成员数量回调
 
- 备注：
  `getRoomMemberCountCBType`类型定义如下：

  ``` obj-c
  typedef void(^getRoomMemberCountCBType)(YIMErrorcodeOC errorcode, NSString* chatRoomID, unsigned int count);
  ```  
   
- 参数：
  `errorcode `：错误码
  `chatRoomID`：频道ID
  `count`：频道中的成员数量 

## 消息管理
游密IM可以进行多种消息类型的交互，比如文本，自定义，表情，图片，语音，文件，礼物等。

### 接收消息 

- 接口：

``` obj-c
- (void) OnRecvMessage:(YIMMessage*) pMessage;
```

- 参数说明：
  `pMessage`：消息
  
- 备注：   
  `YIMMessage` 消息基类成员如下：
  `senderID`： 消息发送者ID
  `receiverID`： 消息接收者ID
  `chatType`：聊天类型，私聊/频道聊天，YIMChatTypeOC
              YIMChatType_Unknow=0,
              YIMChatType_PrivateChat=1  //私聊
              YIMChatType_RoomChat=2    //聊天室
              YIMChatType_Multi=3
  `messageID`： 消息ID
  `messageBody`：消息体，YIMMessageBodyBase
  `createTime`： 消息收发时间 
  `distance`： 若是附近频道的消息，此值是该消息用户与自己的地理位置距离
  
  `YIMMessageBodyBase`成员如下：  
  `messageType`： 消息类型，枚举类型，YIMMessageBodyTypeOC
   YIMMessageBodyType_Unknow = 0,         //未知类型
	 YIMMessageBodyType_TXT = 1,            //文本消息
	YIMMessageBodyType_CustomMesssage = 2, //自定义消息
	YIMMessageBodyType_Emoji = 3,          //表情
	YIMMessageBodyType_Image = 4,          //图片
	YIMMessageBodyType_Voice = 5,          //语音
	YIMMessageBodyType_Video = 6,          //视频
	YIMMessageBodyType_File = 7,           //文件
	YIMMessageBodyType_Gift = 8            //礼物  
	    
  `YIMMessageBodyText`继承`YIMMessageBodyBase`类，类成员如下：
  `messageContent`： 文本消息内容，NSString型
  `attachParam`：发送文本附加信息，NSString型
  
  `YIMMessageBodyCustom`继承`YIMMessageBodyBase`类，类成员如下：
  `messageContent`： 自定义消息内容，NSData型
  
  `YIMMessageBodyFile`继承`YIMMessageBodyBase`类，类成员如下：
  `fileName`： 文件名，NSString型
  `fileType`： 文件类型，枚举类型，YIMFileTypeOC
               YIMFileType_Other = 0   //其它
               YIMFileType_Audio = 1   //音频（语音）
               YIMFileType_Image = 2   //图片
               YIMFileType_Video = 3   //视频
  `fileSize`： 文件大小，unsigned int型
  `fileExtension`： 文件扩展信息，NSString型
  `extParam`： 附加信息，NSString型
  
  `YIMMessageBodyGift`继承`YIMMessageBodyBase`类，类成员如下：
  `anchor`： 主播ID，NSString型
  `giftID`： 礼物ID，int型
  `giftCount`： 礼物数量，unsigned int型
  `extParam`： 附加参数，NSString型
  
  `YIMMessageBodyAudio`继承`YIMMessageBodyBase`类，类成员如下：
  `textContent`： 若使用的是语音转文字录音，此值为语音识别的文本内容，否则是空，NSString型
  `audioTime`： 语音时长（单位：秒），unsigned int型
  `customContent`： 发送语音时的附加参数，NSString型
  `fileSize`： 语音文件大小（单位：字节），unsigned int型
    
- 示例：

```obj-c
(void) OnRecvMessage:(YIMMessage*) pMessage{
    //私聊消息 还是 房间消息的判断
    BOOL isPrivateMessage =( pMessage.chatType ==YIMChatType_PrivateChat) ;

    if(pMessage.messageBody.messageType == YIMMessageBodyType_TXT){
        //收到的是文本消息
        YIMMessageBodyText *tMessage = (YIMMessageBodyText *)pMessage.messageBody;
        NSLog(@"文字消息:%@",[tMessage messageContent]);
    }else if(pMessage.messageBody.messageType == YIMMessageBodyType_Voice){
        //收到的是语音消息
        YIMMessageBodyAudio *vMessage = (YIMMessageBodyAudio *)pMessage.messageBody;
        NSLog(@"文字识别结果:%@,fileSize:%d,audioTime:%d",[vMessage textContent], [vMessage fileSize], [vMessage audioTime] );
        //下载语音文件
        [[YIMClient GetInstance] DownloadAudio:pMessage.messageID strSavePath:@"path"];
    }
    else if (pMessage.messageBody.messageType == YIMMessageBodyType_File ){
        //收到文件
        YIMMessageBodyFile* fileMsg = (YIMMessageBodyFile *)pMessage.messageBody;
        NSLog(@"获得文件:" );

        //下载文件
        [[YIMClient GetInstance] DownloadAudio:pMessage.messageID strSavePath:@"path"];
    }
    else if  (pMessage.messageBody.messageType == YIMMessageBodyType_Gift ){
        //收到礼物
        YIMMessageBodyGift* giftMsg = (YIMMessageBodyGift *)pMessage.messageBody;

        NSLog(@"获得礼物:%d,%d,anchor:%@,extraParam:%@", giftMsg.giftID, giftMsg.giftCount, giftMsg.anchor, giftMsg.extraParam );
    }
    else if ( pMessage.messageBody.messageType == YIMMessageBodyType_CustomMesssage ){
        //自定义消息
        YIMMessageBodyCustom* customMsg = (YIMMessageBodyCustom *)pMessage.messageBody;
        const char* cp = (const char*)customMsg.messageContent.bytes;
        int size = customMsg.messageContent.length;

        NSLog(@"获得自定义消息,size%d", size  );
    }
}
```

### 发送文本消息

- 接口：

  ``` obj-c
  -(unsigned long long) SendTextMessage:(NSString *)receiverID chatType:(YIMChatTypeOC)chatType msgContent:(NSString *) msgContent attachParam:(NSString *)attachParam callback:(sendMessageStatusCBType)callback;
  ```

- 参数说明：
  `receiverID`:接收者ID（用户ID或者频道ID）
  `chatType`: 聊天类型，私聊用`YIMChatType_PrivateChat`，频道用`YIMChatType_RoomChat`
  `msgContent`: 消息内容字符串  
  `attachParam`：发送文本附加信息
  `callback`： 发送文本消息回调
  
- 返回值：
  消息序列号，用于校验一条消息发送成功与否的标识
 
- 备注：
  `sendMessageStatusCBType`类型定义如下：

``` obj-c
typedef void(^sendMessageStatusCBType)(YIMErrorcodeOC errorcode, unsigned int sendTime, bool isForbidRoom,int reasonType,unsigned long long forbidEndTime);
```

- 参数：  
  `errorcode`：错误码
  `sendTime`：消息发送时间
  `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
  `forbidEndTime`：若在频道被禁言，禁言结束时间

### 群发文本消息
用于群发文本消息的接口，每次不要超过200个用户。

``` obj-c
-(YIMErrorcodeOC) MultiSendTextMessage:( NSArray * )receivers  text:(NSString *) text;
```

- 参数说明：
  `receivers`：接受消息的用户id列表
  `text`：文本消息内容
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)
    
### 发送礼物消息
给主播发送礼物消息的接口，支持在游密主播后台查看礼物消息的详细信息和统计信息。客户端还是通过`OnRecvMessage`接收消息。

- 接口：

``` obj-c
 -(unsigned long long) SendGift:(NSString*)anchorID channel:(NSString*)channel giftId:(int)giftId giftCount:(int)giftCount extraParam:(NSString*) extraParam callback:(sendMessageStatusCBType)callback;
```

- 参数说明：
  `anchorID`：主播id
  `channel`：主播所进入的频道ID，通过JoinChatRoom进入频道的用户可以接收到消息。
  `giftId`：礼物物品id，特别的是`0`表示只是留言
  `giftCount`：礼物物品数量
  `extraParam`：扩展内容json字符串，目前要求包含如下字段：
{"nickname":"","server_area":"","location":"","score":"","level":"","vip_level":"","extra":""}  
  `callback`： 发送礼物消息回调
  
- 返回值：
  消息序列号，用于校验一条消息发送成功与否的标识
 
- 备注：
  `sendMessageStatusCBType`类型定义如下：

``` obj-c
typedef void(^sendMessageStatusCBType)(YIMErrorcodeOC errorcode, unsigned int sendTime, bool isForbidRoom,int reasonType,unsigned long long forbidEndTime);
```

- 参数：  
  `errorcode`：错误码
  `sendTime`：文件发送时间
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它 
  `forbidEndTime`：若在频道被禁言，禁言结束时间 

### 发送自定义消息
- 接口：

``` obj-c
-(unsigned long long) SendCustomMessage:(NSString*) receiverID chatType:(YIMChatTypeOC)chatType content:(NSData*)content callback:(sendMessageStatusCBType)callback;
```

- 参数说明：
  `receiverID`：接收者（用户ID或者频道ID）
  `chatType`：聊天类型
  `content`：自定义消息内容  
  `callback`： 发送自定义消息回调
  
- 返回值：
  消息序列号，用于校验一条消息发送成功与否的标识
 
- 备注：
  `sendMessageStatusCBType`类型定义如下：

``` obj-c
typedef void(^sendMessageStatusCBType)(YIMErrorcodeOC errorcode, unsigned int sendTime, bool isForbidRoom,int reasonType,unsigned long long forbidEndTime);
```

- 参数：  
  `errorcode`：错误码
  `sendTime`：文件发送时间
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它 
  `forbidEndTime`：若在频道被禁言，禁言结束时间 

### 发送文件消息
#### 发送文件 

- 接口：

``` obj-c
-(unsigned long long) SendFile:(NSString*) receiverID chatType:(YIMChatTypeOC)chatType filePath:(NSString*)filePath extraParam:(NSString*)extraParam fileType:(YIMFileTypeOC)fileType callback:(sendMessageStatusCBType)callback;
```

- 参数说明：
  `receiverID`：接收者（用户ID或者频道ID）
  `chatType`：聊天类型
  `filePath`：要发送文件的路径  
  `extraParam`：扩展内容，格式自己定义，自己解析
  `fileType`：文件类型，枚举类型，
              YIMFileType_Other=0 //其它
              YIMFileType_Audio=1 //音频
              YIMFileType_Image=2 //图片
              YIMFileType_Video=3 //视频
  `callback`： 发送文件消息回调
  
- 返回值：
  消息序列号，用于校验一条消息发送成功与否的标识
 
- 备注：
  `sendMessageStatusCBType`类型定义如下：

``` obj-c
typedef void(^sendMessageStatusCBType)(YIMErrorcodeOC errorcode, unsigned int sendTime, bool isForbidRoom,int reasonType,unsigned long long forbidEndTime);
```

- 参数：  
  `errorcode`：错误码
  `sendTime`：文件发送时间
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它 
  `forbidEndTime`：若在频道被禁言，禁言结束时间
  
#### 下载文件 
接收消息接口参考收消息通过GetMessageType()分拣出文件消息类型：YIMMessageBodyType_File，用messageID获得消息ID。然后调用函数DownloadAudio下载文件。

- 原型：

``` obj-c
-(void) DownloadAudio:(unsigned long long) ulSerial  strSavePath:(NSString*)strSavePath callback:(downloadCBType)callback;
```

- 参数：
  `ulSerial`：消息ID
  `savePath`：指定保存路径，且为带文件名的完整路径，如果目录不存在，SDK会自动创建，必须保证该路径有可写权限。
   `callback`： 下载文件回调

- 备注：
  `downloadCBType`类型定义如下：

``` obj-c
typedef void(^downloadCBType)(YIMErrorcodeOC errorcode, YIMMessage* msg, NSString* savePath);
```

- 参数：
  `errorcode`：下载结果错误码
  `msg`：消息，`YIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注
  `savePath`：保存路径  
    
### 语音消息管理   
语音消息聊天简要流程：

- 调用IM的语音发送接口就开始录音；
- 调用结束录音接口后自动停止录音并发送。

接收方接收语音消息通知后，调用方控制是否下载，调用下载接口就可以获取录音内容。然后开发者调用播放接口播放wav音频文件。

#### 设置是否自动下载语音消息
SetDownloadAudioMessageSwitch()在初始化之后，启动语音之前调用;若设置了自动下载语音消息，不需再调用DownloadAudio()接口，收到语音消息时会自动下载，自动下载完成也会收到DownloadAudio()接口对应的OnDownload()回调。

- 原型：

  ``` obj-c
  - (YIMErrorcodeOC) SetDownloadAudioMessageSwitch:(bool)download;
  ```    
    
- 参数：
  `download `：true-自动下载语音消息，false-不自动下载语音消息(默认) 
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)
  
#### 设置语音识别的语言
`SendAudioMessage`提供语音识别功能，将语音识别为文字，默认输入语音为普通话，识别文字为简体中文，可通过`SetSpeechRecognizeLanguage`函数设置语言；若需要识别为繁体中文，需要联系我们配置此服务。

- 原型：

  ``` obj-c
  - (YIMErrorcodeOC) SetSpeechRecognizeLanguage:(SpeechLanguageOC) language;
  ```

- 参数：
  `language`：语言，枚举类型，
       YIMSPEECHLANG_MANDARIN=0 //普通话
       YIMSPEECHLANG_YUEYU=1    //粤语
       YIMSPEECHLANG_SICHUAN=2  //四川话（windows不支持）
       YIMSPEECHLANG_HENAN=3    //河南话（只支持iOS）
       YIMSPEECHLANG_ENGLISH=4  //英语
       YIMSPEECHLANG_TRADITIONAL=5 //繁体中文
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义) 
  
#### 发送语音消息  
##### 启动录音
启动录音后，麦克风就处于录音状态，最长60s（超过1分钟就自动发出去），再配合发送录音（StopAndSendAudioMessage）或者取消录音（CancleAudioMessage）的接口使用。

- 接口：

``` obj-c
-(unsigned long long) StartRecordAudioMessage:(NSString *)receiverID chatType:(YIMChatTypeOC)chatType recognizeText:(bool)recognizeText isOpenOnlyRecognizeText:(bool)isOpenOnlyRecognizeText callback:(sendAudioMsgStatusCBType)callback startSendCallback:(startSendAudioMsgCBType)startSendCallback;
```

- 参数说明：
  `receiverID`：接收者 用户ID 或者 频道ID，私聊传入userid，频道聊天传入roomid
  `chatType`：聊天类型，私聊用`YIMChatType_PrivateChat`，频道用`YIMChatType_RoomChat`  
  `recognizeText`：是否带文字识别
  `IsOpenOnlyRecognizeText`：是否开启仅识别语音文本，不发送语音消息
  `callback`： 录音回调

- 返回值：
  消息序列号，用于校验一条消息发送成功与否的标识

- 备注：
  `sendAudioMsgStatusCBType`和`startSendAudioMsgCBType`类型定义如下：

``` obj-c
// sendAudioMsgStatusCBType 发送语音结果回调，自己的语音消息发送成功或者失败的通知。
typedef void(^sendAudioMsgStatusCBType)(YIMErrorcodeOC errorcode, NSString* text, NSString* audioPath, unsigned int audioTime, unsigned int sendTime, bool isForbidRoom, int reasonType, unsigned long long forbidEndTime);

// startSendAudioMsgCBType 开始上传语音回调
typedef void(^startSendAudioMsgCBType)(YIMErrorcodeOC errorcode, NSString* text, NSString* audioPath, unsigned int audioTime);
```

- 参数： 
  `errorcode`：错误码，`YouMeIMCode_Success` 或者 `YouMeIMCode_PTT_ReachMaxDuration` 均表示成功
  `text`：语音转文字识别的文本内容，如果没有用带语音转文字的接口，该字段为空字符串
  `audioPath`：录音生成的wav文件的本地完整路径
  `audioTime`：录音时长(单位秒)    
  
  `sendTime`：语音发送时间  
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言（errorcode==YouMeIMCode_ForbiddenSpeak(被禁言)才有效）    
  `reasonType`：禁言原因，参见ForbidSpeakReason（errorcode==YouMeIMCode_ForbiddenSpeak(被禁言)才有效）    
  `forbidEndTime`：禁言截止时间戳（errorcode==YouMeIMCode_ForbiddenSpeak(被禁言)才有效）
  
***如果只识别语音文本，不发送语音消息，收到以下回调：***
``` obj-c
  - (void)OnGetRecognizeSpeechText:(unsigned long long) iRequestID errorcode:(YIMErrorcodeOC)errorcode text:(NSString*)text;
```    
    
- 回调参数：
  `iRequestID`：消息ID
  `errorcode `：错误码
  `text `：返回的语音识别的文本
  
##### 结束录音并发送

- 接口：

``` obj-c
-(YIMErrorcodeOC) StopAndSendAudioMessage:(NSString *)attachMsg;
```

- 参数说明：
  `attachMsg`：透传消息字符串，可以用来附加一些用户的其他属性，格式自定义，自己解析 （json、xml...），可为空字符串。

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 取消本次录音

- 接口：

``` obj-c
-(YIMErrorcodeOC) CancleAudioMessage;
```

- 参数说明：
  无。

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)

- 回调：
  无。

#### 接收语音消息 
接收消息接口参考收消息通过GetMessageType()分拣出语音消息类型：YIMMessageBodyType_Voice，用messageID获得消息ID。然后调用函数DownloadAudio下载语音消息，下载完成后调用方播放。

- 原型：

``` obj-c
-(void) DownloadAudio:(unsigned long long) ulSerial  strSavePath:(NSString*)strSavePath callback:(downloadCBType)callback;
```

- 参数：
  `ulSerial`：消息ID
  `savePath`：指定保存路径，且为带文件名的完整路径，如果目录不存在，SDK会自动创建，必须保证该路径有可写权限。
  `callback`：下载语音回调

- 备注：
  `downloadCBType`类型定义如下：

``` obj-c
typedef void(^downloadCBType)(YIMErrorcodeOC errorcode, YIMMessage* msg, NSString* savePath);
```

- 参数：
  `errorcode`：下载结果错误码
  `msg`：消息，`YIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注
  `savePath`：保存路径
  
#### 语音播放
##### 播放语音
可以使用SDK内置的播放接口进行语音播放，目前只支持WAV格式。

- 接口：

``` obj-c
- (void) StartPlayAudio:(NSString* ) path callback:(playCompleteCBType)callback;
```

- 参数说明：
  `path`：音频文件绝对路径
  `callback`：播放结果回调

- 备注：
  `playCompleteCBType`类型定义如下：

``` obj-c
typedef void(^playCompleteCBType)(YIMErrorcodeOC errorcode, NSString* path);
```

- 参数：
  `errorcode`：错误码，errorcode == YouMeIMCode_Success表示正常播放结束，errorcode == YouMeIMCode_PTT_CancelPlay表示播放到中途被打断。 
  `path`：被播放的音频文件地址。  

##### 停止语音播放

- 接口：

``` obj-c
- (YIMErrorcodeOC) StopPlayAudio;
```

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)
    
##### 设置语音播放音量
设置语音播放的音量大小。

- 接口：

``` obj-c
- (void) SetVolume:(float) volume;
```

- 参数：
  `volume`：音量值，取值范围`0.0f` - `1.0f`，默认值为 `1.0f`。
    
#### 录音缓存
##### 设置录音缓存目录 
设置录音时用于保存录音文件的缓存目录，如果没有设置，SDK会在APP默认缓存路径下创建一个文件夹用于保存音频文件。

- 接口：

  ``` obj-c
  +(void) SetAudioCacheDir:(NSString*) path;
  ```

- 参数说明：
  `path`：缓存目录绝对路径，** 需要先创建好该目录 **。

- 返回：
  无。

##### 获取当前设置的录音缓存目录

- 接口：

``` obj-c
- (NSString*) GetAudioCachePath;
```

- 返回值：
  返回当前设置的录音缓存目录的完整路径。

##### 清空录音缓存目录
清空当前设置的录音缓存目录。

- 接口：

``` obj-c
- (bool) ClearAudioCachePath;
```

- 返回值：
  `true`：表示清理成功，`false`：表示清理操作失败。
  
#### 设置下载保存目录 
下载保存目录是DownloadAudio的默认下载目录，对下载语音消息和文件均适用；设置下载保存目录在初始化之后，启动语音之前调用，若设置了下载保存目录，调用DownloadAudio()时其中的savePath参数可以为空字符串。

- 原型：

  ``` obj-c
  - (YIMErrorcodeOC) SetDownloadDir:(NSString*) path;
  ```
  
- 参数：
  `path`：下载目录路径，必须保证该路径可写。
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 获取麦克风状态  
这是一个异步操作，操作结果会通过回调参数返回。（该接口会临时占用麦克风）
- 原型：

``` obj-c
- (void) GetMicrophoneStatus:(getMicStatusCBType)callback;
```

- 参数：
  `callback`：获取麦克风状态回调。
  
- 备注：
  `getMicStatusCBType`类型定义如下：

``` obj-c
typedef void(^getMicStatusCBType)(AudioDeviceStatusOC status);
``` 
  
- 参数：
  `status`：麦克风状态值，枚举类型，
            YIMSTATUS_AVAILABLE=0    //可用
            YIMSTATUS_NO_AUTHORITY=1 //无权限
            YIMSTATUS_MUTE=2         //静音
            YIMSTATUS_UNAVAILABLE=3  // 不可用  
    
### 录音上传
#### 启动录音
该接口启动录音

``` obj-c
-(unsigned long long) StartAudioSpeech:(bool)translate callback:(uploadSpeechStatusCBType)callback;
```

- 参数说明：  
  `translate`：是否进行语音转文字识别
  `callback`：录音回调
  
- 返回值：
  消息序列号，用于校验一条消息发送成功与否的标识

- 备注：
  `uploadSpeechStatusCBType`类型定义如下：

``` obj-c
typedef void(^uploadSpeechStatusCBType)(YIMErrorcodeOC errorcode,  AudioSpeechInfo* audioSpeechInfo);
```

- 参数：
  `errorcode`：错误码
  `audioSpeechInfo`：音频文件信息  

  `AudioSpeechInfo`结构体成员如下： 
  `requestID`：消息ID 
  `downloadURL`：语音文件的下载地址
  `audioTime`：录音时长，单位秒
  `fileSize`：文件大小，字节  
  `textContent`：语音识别结果，可能为空null or ""

- 相关函数：

  ``` obj-c
  //取消录音
  -(YIMErrorcodeOC) CancleAudioMessage;
  //停止录音
  -(YIMErrorcodeOC) StopAudioSpeech;
  ```

#### 停止录音并上传
该接口对应StartAudioSpeech()。
该接口自动上传并只返回音频文件的下载链接，下载链接指向的是AMR格式的音频文件，不会自动发送。

- 接口

``` obj-c
-(YIMErrorcodeOC) StopAudioSpeech;
```

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)

#### 根据url下载录音文件
- 接口

``` obj-c
-(void) DownloadAudioByUrl:(NSString*) downloadURL  strSavePath:(NSString*)strSavePath callback:(downloadByUrlCBType)callback;
```

- 参数
  `downloadURL`：语音文件的url地址，OnStopAudioSpeechStatus返回的downloadURL地址。
  `strSavePath`：语音文件的本地存放地址,带文件名的全路径
  `callback`：下载录音文件回调
  
- 备注：
  `downloadByUrlCBType`类型定义如下：

``` obj-c
typedef void(^downloadByUrlCBType)(YIMErrorcodeOC errorcode, NSString* strFromUrl, NSString* savePath);
```  

- 参数：
  `errorcode`：错误码
  `strFromUrl`：下载的语音文件url
  `strSavePath`：本地存放地址
        
### 消息屏蔽
#### 屏蔽/解除屏蔽用户消息
若屏蔽用户的消息，此屏蔽用户发送的私聊/频道消息都接收不到。

- 原型：

  ``` obj-c
  -(void) BlockUser:(NSString*) userID block:(bool) block callback:(blockUserCBType)callback;
  ```
  
- 参数：
  `userID `：用户ID
  `block `：true-屏蔽 false-解除屏蔽
  `callback`：屏蔽用户回调
  
- 备注：
  `blockUserCBType`类型定义如下：

``` obj-c
typedef void(^blockUserCBType)(YIMErrorcodeOC errorcode, NSString* userID, bool block);
```    

- 参数：
  `errorcode `：错误码
  `userID `：用户ID
  `block `：true-屏蔽 false-解除屏蔽    

#### 解除所有已屏蔽用户
- 原型：

  ``` obj-c
  -(void) UnBlockAllUser:(unBlockAllUserCBType)callback;
  ```

- 参数：  
  `callback`：解除所有屏蔽回调
  
- 备注：
  `unBlockAllUserCBType`类型定义如下：

``` obj-c
typedef void(^unBlockAllUserCBType)(YIMErrorcodeOC errorcode);
```     

- 参数：
  `errorcode `：错误码
  
#### 获取被屏蔽消息用户
- 原型：

  ``` obj-c
  -(void) GetBlockUsers:(getBlockUsersCBType)callback;
  ```
  
- 参数：  
  `callback`：获取屏蔽用户回调
  
- 备注：
  `getBlockUsersCBType`类型定义如下：

``` obj-c
typedef void(^getBlockUsersCBType)(YIMErrorcodeOC errorcode, NSArray* userList);
```     
    
- 参数：
  `errorcode`：错误码
  `userList`：屏蔽用户ID列表, NSArray<NString*> 

### 消息功能设置
#### 设置消息已读
将消息标为已读。

- 接口：

``` obj-c
- (YIMErrorcodeOC) SetMessageRead:(unsigned long long)msgID read:(bool)read;
```  

- 参数：
  `msgID`：消息ID
  `read`：消息是否已读，true为已读，false为未读。 
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)   

#### 切换获取消息模式
在自动接收消息和手动接收消息间切换，默认是自动接收消息。

- 接口：

``` obj-c
// 是否自动接收消息(true:自动接收(默认)   false:不自动接收消息，
// 有新消息达到时，SDK会发出OnReceiveMessageNotify回调，
// 调用方需要调用GetMessage获取新消息)
-(void) SetReceiveMessageSwitch:( NSArray * )targets receive:(bool)receive;
```

- 参数：
  `targets`：频道ID数组
  `receive`：true为自动接收消息，false为手动接收消息，默认为true。
    
#### 手动获取消息
在手动接收消息模式，需要调用该接口后才能收到 `OnRecvMessage` 通知。

- 接口：
``` obj-c
-(YIMErrorcodeOC) GetNewMessage:( NSArray * )targets;
```

- 参数：
  `targets`：频道ID数组

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)

#### 新消息通知
在手动接收消息模式，会通知有新消息。

- 接口：

``` obj-c
- (void) OnReceiveMessageNotify:(YIMChatTypeOC)chatType targetID:(NSString*)targetID {}
```

- 参数：
  `chatType`：聊天类型
  `targetID`：聊天对象，私聊为UserID，频道为频道ID

### 消息记录管理
#### 设置是否保存频道聊天记录
设置是否保存频道消息记录（默认不保存）。`私聊历史记录默认保存`。

- 接口：

``` obj-c
-(void) SetRoomHistoryMessageSwitch:( NSArray * )targets save:(bool)save;
```

- 参数：
  `targets`：频道ID数组，指定要保存本地历史记录的频道ID
  `save`：true为保存聊天记录，false为不保存，默认false。
    
#### 拉取频道最近聊天记录
从服务器拉取频道最近的聊天历史记录。
这个功能默认不开启，需要的请联系我们修改服务器配置。联系我们，可以通过专属游密支持群或者技术支持的大群。

- 接口：

``` obj-c
-(void) QueryRoomHistoryMessageFromServer:(NSString*)roomID count:(int)count direction:(int)direction callback:(queryRoomHistoryMsgCBType)callback;
```

- 参数说明：
  `roomID`：字符串,频道id
  `count`：消息数量(最大200条)
  `directon`：历史消息排序方向 0：按时间戳升序	1：按时间戳逆序
  `callback`：查询频道聊天记录回调
  
- 备注：
  `queryRoomHistoryMsgCBType`类型定义如下：

``` obj-c
typedef void(^queryRoomHistoryMsgCBType)(YIMErrorcodeOC errorcode, NSString* roomID, int remain, NSArray* messageList);
```   

- 参数：   
  `errorcode`：错误码
  `roomID`： 房间ID
  `remain`： 剩余消息数量
  `messageList`： 消息列表(NSArray<YIMMessage>)，`YIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注

#### 本地历史记录管理
##### 查询本地聊天历史记录
- 接口：

``` obj-c
-(void) QueryHistoryMessage:(NSString*) targetID chatType:(YIMChatTypeOC) chatType startMessageID:(unsigned long long)startMessageID count:(int)count direction:(int)direction callback:(queryHistoryMsgCBType)callback;
```

- 参数说明：
  `targetID`：私聊用户的id或者频道id
  `chatType`：聊天类型，私聊／频道聊天
  `startMessageID`：起始历史记录消息id（与requestid不同），为0表示首次查询，将倒序获取count条记录
  `count`：最多获取多少条
  `direction`：历史记录查询方向，`startMessageID=0`时,direction使用默认值0；startMessageID>0时，0表示查询比startMessageID小的消息，1表示查询比startMessageID大的消息
  `callback`：查询本地聊天记录回调
  
- 备注：
  `queryHistoryMsgCBType`类型定义如下：

``` obj-c
typedef void(^queryHistoryMsgCBType)(YIMErrorcodeOC errorcode, NSString* targetID, int remain, NSArray* messageList);
```   

- 参数：
  `errorcode`：错误码
  `targetID`：用户ID/频道ID
  `remain`：剩余历史消息条数
  `messageList`：消息列表（NSArray<YIMMessage*>），`YIMMessage`消息基类详细，查看 [接收消息](#接收消息)备注
  
- 相关函数：
  对于房间本地记录，需要先设置自动保存房间消息。`SetRoomHistoryMessageSwitch`

##### 根据时间清理本地聊天历史记录
建议定期清理本地历史记录。

- 接口：

``` obj-c
-(YIMErrorcodeOC) DeleteHistoryMessage:(YIMChatTypeOC)chatType time:(unsigned long long)time;
```

- 参数说明：
  `chatType`：YIMChatType:私聊消息／房间消息
  `time`：Unix timestamp,精确到秒，表示删除这个时间点之前的所有历史记录
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 根据消息ID清理本地聊天历史记录
- 接口：

``` obj-c
-(YIMErrorcodeOC) DeleteHistoryMessageByID:(unsigned long long)messageID;
```

- 参数说明：
  `messageID`：如果指定了大于0的值，将删除指定消息id的历史记录（查询本地历史记录时，消息里给的ID）
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)  
    
##### 根据用户ID清理本地私聊历史记录
可以根据用户ID删除对应的本地私聊历史记录,保留消息ID白名单中的消息记录,白名单列表为空时删除与该用户ID间的所有本地私聊记录。

- 接口：

``` obj-c
- (YIMErrorcodeOC) DeleteSpecifiedHistoryMessage:(NSString*)targetID chatType:(YIMChatTypeOC)chatType excludeMesList:(NSMutableArray *)excludeMesList;
```

- 参数说明：
  `targetID`：用户ID。
  `chatType`：指定清理私聊消息。
  `excludeMesList`：与该用户ID的私聊消息ID白名单。
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 获取最近私聊联系人列表
该接口是根据本地历史消息记录生成的最近联系人列表，按最后聊天时间倒序排列。该列表会受清理历史记录消息的接口影响。

- 接口：

``` obj-c
-(void) GetRecentContacts:(getRecentContactsCBType)callback;
```

- 参数说明：  
  `callback`：查询最近联系人回调
  
- 备注：
  `getRecentContactsCBType`类型定义如下：

``` obj-c
typedef void(^getRecentContactsCBType)(YIMErrorcodeOC errorcode, NSArray* contactList);
```   

- 参数：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)  
  `contactList`：最近联系人列表 NSArray<YIMContactSessionInfoOC*>
  
  `YIMContactSessionInfoOC`类成员如下：
  `contactID`: 联系人ID
  `messageType`: 消息类型，YIMMessageBodyTypeOC
  `messageContent`: 消息内容
  `createTime`: 消息创建时间
  `notReadMsgNum`: 未读消息数量

### 高级功能
#### 文本翻译
`YIMClient`的`TranslateText`接口将文本翻译成指定语言的文本，异步返回结果通过`YIMCallbackProtocol`的`OnTranslateTextComplete`接口返回。

- 原型

``` obj-c
-(unsigned int) TranslateText:(NSString*)text  destLangCode:(LanguageCodeOC) destLangCode srcLangCode:(LanguageCodeOC) srcLangCode callback:(translateTextCompleteCBType)callback;
```

- 参数  
  `text`：待翻译文本
  `destLangCode`：目标语言编码
  `srcLangCode`：原文本语言编码
  `callback`：文本翻译回调
  
- 返回值：
  请求ID 

- 备注：
  `translateTextCompleteCBType`类型定义如下：

``` obj-c
typedef void(^translateTextCompleteCBType)(YIMErrorcodeOC errorcode, unsigned int requestID, NSString* text, LanguageCodeOC srcLangCode, LanguageCodeOC destLangCode);
```  

- 参数：
  `errorcode`：等于0才是操作成功
  `requestID`：请求ID
  `text`：翻译后的文本
  `srcLangCode`：源语言编码
  `destLangCode`：目标语言编码
      
#### 敏感词过滤 
考虑到客户端可能需要传递一些自定义消息，关键字过滤方法就直接提供出来，客户端可以选择是否过滤关键字，并且可以根据匹配到的等级能进行广告过滤。比如`level`返回的值为'2'表示广告，那么可以在发送时选择不发送，接收方可以选择不展示。

- 接口：

  ``` obj-c
  +(NSString*) filterKeyWorks:(NSString*)originContent  level:(int*) level;
  ```

- 参数说明：
  `originContent`：原字符串。
  `level`：匹配的策略词等级，默认为0，`0`表示正常，其余值是按需求可配

- 返回：
  过滤关键字后的字符串，敏感词替换为"**"

## 游戏暂停与恢复
### 游戏暂停通知
建议游戏放入后台时通知该接口，以便于得到更好重连效果。
调用OnPause(false),在游戏暂停后依旧接收IM消息；
调用OnPause(true),在游戏恢复运行时会主动拉取暂停期间未接收的消息，收到OnRecvMessage()回调。

- 接口：

``` obj-c
-(void) OnPause:(bool)pauseReceiveMessage;
```

- 参数：
  `pauseReceiveMessage`：是否暂停接收IM消息，true-暂停接收 false-不暂停接收

### 游戏恢复运行通知
建议游戏从后台激活到前台时通知该接口，以便于得到更好重连效果。

- 接口：
``` obj-c
-(void) OnResume;
```

## 用户举报和禁言 
### 用户举报
`Accusation`接口提供举报功能，对用户的违规行为进行举报，管理员在后台进行审核处理并将结果通知用户。

- 原型

``` obj-c
- (YIMErrorcodeOC) Accusation:(NSString*)userID source:(YIMChatTypeOC)source reason:(int)reason description:(NSString*)description extraParam:(NSString*)extraParam
```

- 参数：
  `userID`：被举报用户ID
  `source`：来源（私聊 频道）
  `reason`：原因，由用户自定义
  `description`：原因描述
  `extraParam`：附加信息JSON格式 ({"nickname":"","server_area":"","level":"","vip_level":""}）
  
- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)  

- 举报处理结果通知
当管理员对举报进行审核处理后，会将举办处理结果通知用户

- 原型

``` obj-c
- (void)OnAccusationResultNotify:(AccusationDealResultOC)result userID:(NSString*)userID accusationTime:(unsigned int)accusationTime
```

- 回调参数：
  `result`：处理结果，枚举类型，
  YIMACCUSATIONRESULT_IGNORE=0 // 忽略   
  YIMACCUSATIONRESULT_WARNING=1 // 警告
  YIMACCUSATIONRESULT_FROBIDDEN_SPEAK=2 // 禁言     
  `userID`：被举报用户ID
  `accusationTime`：举报时间
  
### 禁言查询
查询所在频道的禁言状态。

- 原型：

  ``` obj-c
  - (void) GetForbiddenSpeakInfo:(getForbiddenSpeakInfoCBType)callback;
  ```

- 参数说明：  
  `callback`：禁言查询回调
  
- 备注：
  `getForbiddenSpeakInfoCBType`类型定义如下：

``` obj-c
typedef void(^getForbiddenSpeakInfoCBType)(YIMErrorcodeOC errorcode, NSArray* vecForbiddenSpeakInfos);
```   

- 参数：
  `errorcode`：错误码
  `forbiddenSpeakArray`：禁言状态列表,每一个代表一个频道的禁言状态  NSArray<ForbiddenSpeakInfoOC*>

  `ForbiddenSpeakInfoOC` 类成员如下：
  `channelID`：频道ID
  `isForbidRoom`：此频道是否被禁言，true-被禁言，false-未被禁言
  `reasonType`：禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
  `endTime`：若被禁言，禁言结束时间
   
## 公告管理
基本流程：申请开通公告功能->后台添加新公告然后设置公告发送时间,公告消息类型,发送时间,接收公告的频道等。->(客户端流程) 设置对应监听-> 调用对应接口->回调接收
### 公告功能简述

1.公告发送由后台配置，如类型、周期、发送时间、内容、链接、目标频道、次数、起始结束时间等。

2.公告三种类型：跑马灯，聊天框，置顶公告，
(1) 跑马灯，聊天框公告可设置发送时间，次数和间隔（从指定时间点开始隔固定间隔时间发送多次，界面展示及显示时长由客户端决定）；
(2) 置顶公告需设置开始和结束时间（该段时间内展示）。

3.三种公告均有一次性、周期性两种循环属性：
  一次性公告，到达指定时间点，发送该条公告；
  周期性公告，跟一次性公告发送规则一致，但是可以设置发送周期（在每周哪几天的指定时间发送）。
  
4.跑马灯与聊天框公告只有发送时间点在线的用户才能收到该公告，显示规则由客户端自己决定，两者区别主要是界面显示的区分。

5.置顶公告有显示起始和结束时间，表示该时段内显示，公告发送时间点在线的用户会收到该公告，公告发送时间点未在线用户，在公告显示时段登录，登录后可通过查询公告接口查到该公告。

6.公告撤销
仅针对置顶公告，公告显示时段撤销公告，客户端会收到公告撤销通知，界面进行更新。

### 接收公告
管理员在后台发布公告，当到达指定时间会收到该公告，界面根据不同类型的公告进行展示。

- 原型：

  ``` obj-c
  - (void) OnRecvNotice:(YIMNoticeOC*) notice {}
  ```
  
- 参数：
  `notice`：公告信息
  
- 备注：
  `YIMNoticeOC`结构体成员如下：
  `noticeID`：公告ID，unsigned long long型
  `noticeType`：公告类型，int型
  `channelID`：发布公告的频道ID，NSString型
  `content`：公告内容，NSString型
  `linkText`：公告链接关键字，NSString型
  `linkAddr`：公告链接地址，NSString型
  `beginTime`：公告开始时间，unsigned int型
  `endTime`：公告结束时间，unsigned int型  

### 撤销公告
对于某些类型的公告（如置顶公告），需要在界面展示一段时间，如果管理员在该时间段执行撤销该公告，会收到撤销公告通知，界面进行相应更新。

- 原型：

  ``` obj-c
  - (void) OnCancelNotice:(unsigned long long) noticeID channelID:(NSString*)channelID {}
  ```
  
- 参数：
  `noticeID`：公告ID
  `channelID`：频道ID

### 查询公告
公告在配置的时间点下发到客户端，对于某些类型的公告（如置顶公告）需要在某个时间段显示在，如果用户在公告下发时间点未在线，而在公告展示时间段内登录，应用可根据自己的需要决定是否展示该公告，`QueryNotice`查询公告，结果通过上面的`OnRecvNotice`异步回调返回。

- 原型：

``` obj-c
- (YIMErrorcodeOC) QueryNotice;
```

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)      
   
## 地理位置
### 获取当前的地理位置
可通过`YIMClient`的`GetCurrentLocation`接口请求获取自己当前地理位置。异步返回结果通过`YIMCallbackProtocol`的`OnUpdateLocation`接口返回。

- 接口：

```  obj-c
-(YIMErrorcodeOC) GetCurrentLocation;
```

- 返回值：
  `YIMErrorcodeOC`：错误码，详细描述见[错误码定义](#错误码定义)

- 异步回调接口:

```  obj-c
- (void) OnUpdateLocation:(YIMErrorcodeOC) errorcode  location:(GeographyLocationOC*) location;
```

- 回调参数：
  `errorcode`：等于0才是操作成功。
  `location`：当前地理位置及行政区划信息。
  
- 备注：
  `GeographyLocationOC`结构体成员如下：
  `districtCode`：uint类型，地理区域编码，可根据此参数组合附近频道id
  `country`：NSString类型，国家
  `province`：NSString类型，省份
  `city`：NSString类型，城市
  `districtCounty`：NSString类型，区县
  `street`：NSString类型，街道
  `longitude`：double类型，经度
  `latitude`：double类型，纬度  

### 获取附近的人
获取附近的目标(人 房间) ,若需要此功能，请联系我们开启LBS服务。若已开启服务，此功能生效的前提是自己和附近的人都获取了自己的地理位置，即调用了IM的获取当前地理位置接口。

- 条件：需要开通LBS定位服务才能使用

- 接口：

``` obj-c
-(void) GetNearbyObjects:(int) count  serverAreaID:(NSString*)serverAreaID districtlevel:(YouMeDistrictLevelOC)districtlevel resetStartDistance:(bool)resetStartDistance callback:(getNearbyObjectsCBType)callback;
```

- 参数
  `count`：获取附近的目标数量（一次最大200）
  `serverAreaID`：游戏区服（对应设置用户信息中的区服，如果只需要获得本服务器的，要填；否则填""）
  `districtlevel`：行政区划等级（默认选YouMeDISTRICT_UNKNOW）
  YouMeDISTRICT_UNKNOW=0   //未知
  YouMeDISTRICT_COUNTRY=1  //国家
  YouMeDISTRICT_PROVINCE=2 //省份
  YouMeDISTRICT_CITY=3     //城市
  YouMeDISTRICT_COUNTY=4   //区县
  YouMeDISTRICT_STREET=5   //街道
  `resetStartDistance`：是否重置查找起始距离（是否从距自己0米开始查找）
  `callback`：获取附近的人回调
  
- 备注：
  `getNearbyObjectsCBType`类型定义如下：

``` obj-c
typedef void(^getNearbyObjectsCBType)(YIMErrorcodeOC errorcode, NSArray* neighbourList, unsigned int startDistance, unsigned int endDistance);
```   

- 参数:
  `errorcode`：等于0才是操作成功。
  `neighbourList`：附近的人地理位置信息列表(NSArray<RelativeLocationOC*>)
  `startDistance`：最近距离
  `endDistance`：最远距离  

  `RelativeLocationOC`结构体成员如下：
  `distance`: 与自己的距离
  `longitude`: 经度
  `latitude`: 纬度
  `userID`: 附近用户ID
  `country`: 国家
  `province`: 省份
  `city` 城市
  `districtCounty` 区县
  `street` 街道  

### 地理位置更新
从资源和耗电方面的考虑，SDK不自动监听地理位置的变化，如果调用方有需要可调用`YIMClient`的`SetUpdateInterval`接口，设置更新时间间隔，SDK会按设定的时间间隔监听位置变化并通知上层。（如果应用对地理位置变化关注度不大，最好不要设置自动更新）

- 原型：

``` obj-c
-(void) SetUpdateInterval:(unsigned int) interval;
```

- 参数
  `interval`：时间间隔（单位：分钟）
    
### 获取与指定用户距离
获取与指定用户距离之前，需要调用`getCurrentLocation`成功获取自己的地理位置,指定的用户也调用`getCurrentLocation`成功获取其地理位置。

- 原型：

```obj-c
-(void) GetDistance:(NSString*)userID callback:(getDistanceCBType)callback;
```

- 参数：
  `userID`：用户ID
  `callback`：获取与指定用户距离的回调
  
- 备注：
  `getDistanceCBType`类型定义如下：

``` obj-c
typedef void(^getDistanceCBType)(YIMErrorcodeOC errorcode, NSString* userID, unsigned int distance);
```   

- 参数：
  `errorcode`：错误码
  `userID`: 用户ID
  `distance`: 距离（米）    

## 关系链管理
### 用户信息管理
#### 用户资料变更的通知
当好友的用户资料变更时会收到此通知，使用方根据需要决定是否重新获取资料变更好友的用户信息。

- 原型

  ``` obj-c
  - (void) OnUserInfoChangeNotify:(NSString*) userID;
  ```

- 参数：
  `userID`：资料变更用户的用户ID 
  
#### 设置用户基本资料
设置用户的基本资料，昵称，性别，个性签名，地理位置等。

- 原型

  ``` obj-c
  - (void) SetUserProfileInfo:(IMUserSettingInfoOC*) userInfo callback:(setUserProfileCBType)callback;
  ```

- 参数说明：
  `userInfo`：用户基本信息
  `callback`：设置用户资料回调
  
- 备注：  
  `IMUserSettingInfoOC`结构体成员如下：  
  `nickName`：NSString类型，昵称，长度最大为64bytes
  `sex`：枚举类型，性别，IMUserSexOC: 
                      IM_SEX_UNKNOWN=0  //未知性别
                      IM_SEX_MALE=1     //男性
                      IM_SEX_FEMALE=2   //女性
  `personalSignature`：NSString类型，个性签名，长度最大为120bytes
  `country`：NSString类型，国家  
  `province`：NSString类型，省份
  `city`：NSString类型，城市
  `extraInfo`：NSString类型，扩展信息，JSON格式   
  
  `setUserProfileCBType`类型定义如下：

``` obj-c
typedef void(^setUserProfileCBType)(YIMErrorcodeOC errorcode);
```   
  
#### 查询用户基本资料
查询用户的基本资料，昵称，性别，个性签名，头像，地理位置，被添加权限等。

- 原型

  ``` obj-c
  - (void) GetUserProfileInfo:(NSString*) userID callback:(queryUserProfileCBType)callback;
  ```

- 参数说明：
  `userID`：用户ID
  `callback`：查询用户资料回调
  
- 备注：
  `queryUserProfileCBType`类型定义如下：

``` obj-c
typedef void(^queryUserProfileCBType)(YIMErrorcodeOC errorcode, IMUserProfileInfoOC* userInfo);
```   
  
- 参数：
  `errorcode`：错误码
  `userInfo`：用户资料 

  `IMUserProfileInfoOC`结构体成员如下：  
  `userID`：NSString类型，用户ID
  `photoURL`：NSString类型，头像url
  `onlineState`：枚举类型，在线状态，YIMUserStatusOC:
                 YIMSTATUS_ONLINE=0   //在线，默认值（已登录）
                 YIMSTATUS_OFFLINE=1  //离线
                 YIMSTATUS_INVISIBLE=2 //隐身                
  `beAddPermission`：枚举类型，被好友添加权限，IMUserBeAddPermissionOC:
                   IM_NOT_ALLOW_ADD=0    //不允许被添加
                   IM_NEED_VALIDATE=1    //需要验证
                   IM_NO_ADD_PERMISSION=2 //允许被添加，不需要验证, 默认值  
  `foundPermission`：枚举类型，能否被查找的权限，IMUserFoundPermissionOC:
                    IM_CAN_BE_FOUND=0      //能被其它用户查找到，默认值
                    IM_CAN_NOT_BE_FOUND=1  //不能被其它用户查找到                    
  `settingInfo`：IMUserSettingInfoOC类型，用户基本信息，详见#设置用户信息-备注
  
#### 设置用户头像

- 原型

  ``` obj-c
  - (void) SetUserProfilePhoto:(NSString*) photoPath callback:(setPhotoUrlCBType)callback;
  ```

- 参数：
  `photoPath`：本地图片绝对路径,url长度在500bytes内，图片大小在100kb内
  `callback`：设置用户头像回调
  
- 备注：
  `setPhotoUrlCBType`类型定义如下：

``` obj-c
typedef void(^setPhotoUrlCBType)(YIMErrorcodeOC errorcode, NSString* photoUrl);
```   

- 参数：
  `errorcode`：错误码
  `photoUrl`：图片下载路径  
  
#### 切换用户状态

- 原型

  ``` obj-c
  - (void) SwitchUserStatus:(NSString*) userID  userStatus:(YIMUserStatusOC) userStatus callback:(switchUserOnlineStateCBType)callback;
  ```

- 参数说明：
  `userID`：用户ID
  `userStatus`：用户在线状态，枚举类型，YIMUserStatusOC:
                 YIMSTATUS_ONLINE=0   //在线，默认值（已登录）
                 YIMSTATUS_OFFLINE=1  //离线
                 YIMSTATUS_INVISIBLE=2 //隐身
  `callback`：切换用户状态回调
  
- 备注：
  `switchUserOnlineStateCBType`类型定义如下：

``` obj-c
typedef void(^switchUserOnlineStateCBType)(YIMErrorcodeOC errorcode);
```                  
    
- 参数：
  `errorcode`：错误码 
  
#### 设置好友添加权限

- 原型

  ``` obj-c
  - (void) SetAddPermission:(bool) beFound beAddPermission:(IMUserBeAddPermissionOC) beAddPermission callback:(setUserProfileCBType)callback;
  ```

- 参数说明：
  `beFound`：能否被别人查找到，true-能被查找，false-不能被查找
  `beAddPermission`：被其它用户添加的权限，枚举类型，IMUserBeAddPermissionOC:
                   IM_NOT_ALLOW_ADD=0    //不允许被添加
                   IM_NEED_VALIDATE=1    //需要验证
                   IM_NO_ADD_PERMISSION=2 //允许被添加，不需要验证, 默认值  
  `callback`：设置添加权限回调
  
- 备注：
  `setUserProfileCBType`类型定义如下：

``` obj-c
typedef void(^setUserProfileCBType)(YIMErrorcodeOC errorcode);
```         
  
- 参数：
  `errorcode`：错误码  
  
### 好友管理  
#### 查找好友
查找将要添加的好友，获取该好友的简要信息。

- 原型

  ``` obj-c
  - (void) FindUser:(int) findType target:(NSString*) target callback:(findUserCBType)callback;
  ```

- 参数：
  `findType`：查找类型，0-按ID查找，1-按昵称查找
  `target`：对应查找类型选择的用户ID或昵称
  `callback`：查找好友回调
  
- 备注：
  `findUserCBType`类型定义如下：

``` obj-c
typedef void(^findUserCBType)(YIMErrorcodeOC errorcode, NSArray* users);
```     
  
- 参数：
  `errorcode`：错误码 
  `users`：用户简要信息列表 (NSArray<IMUserBriefInfoOC*>) 

  `IMUserBriefInfoOC`结构体成员如下：  
  `userID`：NSString类型，用户ID
  `nickName`：NSString类型，用户昵称
  `userStatus`：枚举类型，在线状态，YIMUserStatusOC:
                 YIMSTATUS_ONLINE=0   //在线
                 YIMSTATUS_OFFLINE=1  //离线
                 YIMSTATUS_INVISIBLE=2 //隐身     
       
#### 添加好友

- 原型

  ``` obj-c
  - (void) RequestAddFriend:(NSArray *) users comments:(NSString*) comments callback:(requestAddFriendCBType)callback;
  ```

- 参数说明：
  `users`：需要添加为好友的用户ID列表(NSArray<NSString*>)
  `comments`：备注或验证信息，长度最大128bytes
  `callback`：设置添加权限回调
  
- 备注：
  `requestAddFriendCBType`类型定义如下：

``` obj-c
typedef void(^requestAddFriendCBType)(YIMErrorcodeOC errorcode, NSString* userID);
```     
  
- 参数：
  `errorcode`：错误码 
  `userID`：用户ID
  
*** 注意 ***
  被请求方会收到被添加为好友的通知，被请求方如果设置了需要验证才能被添加，会收到下面的回调：
  
  ``` obj-c   
  - (void) OnBeRequestAddFriendNotify:(NSString*) userID comments:(NSString*) comments;
  ```   
  
- 参数：     
  `userID`：请求方的用户ID
  `comments`：备注或验证信息
    
  被请求方如果设置了不需要验证就能被添加，收到下面的回调：
  
  ``` obj-c   
  - (void) OnBeAddFriendNotify:(NSString*) userID comments:(NSString*) comments;
  ```   
  
- 参数：     
  `userID`：请求方的用户ID
  `comments`：备注或验证信息  
  
#### 处理好友请求
当前用户有被其它用户请求添加为好友的请求时，处理添加请求。

- 原型

  ``` obj-c
  - (void) DealBeRequestAddFriend:(NSString*) userID dealResult:(int) dealResult callback:(dealBeRequestAddFriendCBType)callback;
  ```

- 参数：
  `userID`：请求方的用户ID
  `dealResult`：处理结果，0-同意，1-拒绝
  `callback`：处理请求回调
  
- 备注：
  `dealBeRequestAddFriendCBType`类型定义如下：

``` obj-c
typedef void(^dealBeRequestAddFriendCBType)(YIMErrorcodeOC errorcode, NSString* userID, NSString* comments, int dealResult);
```     
  
- 参数：
  `errorcode`：错误码 
  `userID`：请求方的用户ID
  `comments`：备注或验证信息
  `dealResult`：处理结果，0-同意，1-拒绝 
  
*** 注意 ***
 
   如果被请求方设置了需要验证才能被添加为好友，在被请求方成功处理了请求方的好友请求后，请求方能收到添加好友请求结果的通知，收到下面的回调：
   
  ``` obj-c   
  - (void) OnRequestAddFriendResultNotify:(NSString*) userID comments:(NSString*) comments dealResult:(int) dealResult;
  ```
  
- 参数：   
  `userID`：被请求方的用户ID
  `comments`：备注或验证信息
  `dealResult`：处理结果，0-同意，1-拒绝  
    
#### 删除好友

- 原型

  ``` obj-c
  - (void) DeleteFriend:(NSArray *) users deleteType:(int) deleteType callback:(deleteFriendCBType)callback;
  ```

- 参数：
  `users`：需删除的好友用户ID列表(NSArray<NSString*>)
  `deleteType`：删除类型，0-双向删除，1-单向删除(删除方在被删除方好友列表依然存在)
  `callback`：删除好友回调
  
- 备注：
  `deleteFriendCBType`类型定义如下：

``` obj-c
typedef void(^deleteFriendCBType)(YIMErrorcodeOC errorcode, NSString* userID);
```     

  
- 参数：
  `errorcode`：错误码 
  `userID`：被删除好友的用户ID
  
*** 注意 ***
 
   如果删除方采用的是双向删除，被删除方能收到其被好友删除的通知，收到下面的回调：
   
  ``` obj-c   
  - (void) OnBeDeleteFriendNotify:(NSString*) userID;
  ```
  
- 参数：   
  `userID`：删除方的用户ID   
  
#### 拉黑好友

- 原型

  ``` obj-c
  - (void) BlackFriend:(int) type users:(NSArray *) users callback:(blackFriendCBType)callback;
  ```

- 参数说明：
  `type`：拉黑类型，0-拉黑，1-解除拉黑
  `users`：需拉黑的好友用户ID列表(NSArray<NSString*>)
  `callback`：拉黑好友回调
  
- 备注：
  `blackFriendCBType`类型定义如下：

``` obj-c
typedef void(^blackFriendCBType)(YIMErrorcodeOC errorcode, int type, NSString* userID);
```     
 
- 参数：
  `errorcode`：错误码 
  `type`：拉黑类型，0-拉黑，1-解除拉黑
  `userID`：被拉黑方用户ID    
  
#### 查询好友列表
查询当前用户已添加的好友，也能查找被拉黑的好友。

- 原型

  ``` obj-c
  - (void) QueryFriends:(int) type startIndex:(int) startIndex count:(int) count callback:(queryFriendsCBType)callback;
  ```

- 参数：
  `type`：查找类型，0-正常状态的好友，1-被拉黑的好友
  `startIndex`：起始序号，作用为分批查询好友（例如：第一次从序号0开始查询30条，第二次就从序号30开始查询相应的count数）
  `count`：数量（一次最大100） 
  `callback`：查询好友列表回调
  
- 备注：
  `queryFriendsCBType`类型定义如下：

``` obj-c
typedef void(^queryFriendsCBType)(YIMErrorcodeOC errorcode, int type, int startIndex, NSArray* friends);
```      
 
- 参数：
  `errorcode`：错误码 
  `type`：查找类型，0-正常状态的好友，1-被拉黑的好友
  `startIndex`：起始序号
  `friends`：好友列表(NSArray<IMUserBriefInfoOC>) 
  
  `IMUserBriefInfoOC`结构体成员如下：  
  `userID`：NSString类型，用户ID
  `nickName`：NSString类型，用户昵称
  `userStatus`：枚举类型，在线状态，YIMUserStatusOC:
                 YIMSTATUS_ONLINE=0   //在线
                 YIMSTATUS_OFFLINE=1  //离线   
  
#### 查询好友请求列表
查询当前用户收到的被添加请求的好友列表。

- 原型

  ``` obj-c
  - (void) QueryFriendRequestList:(int) startIndex count:(int) count callback:(queryFriendRequestListCBType)callback;
  ```

- 参数：  
  `startIndex`：起始序号，作用为分批查询好友请求（例如：第一次从序号0开始查询10条，第二次就从序号10开始查询相应的count数）
  `count`：数量（一次最大20）  
  `callback`：查询好友请求列表回调
  
- 备注：
  `queryFriendRequestListCBType`类型定义如下：

``` obj-c
typedef void(^queryFriendRequestListCBType)(YIMErrorcodeOC errorcode, int startIndex, NSArray* requestList);
```     
  
- 参数：
  `errorcode`：错误码 
  `startIndex`：起始序号
  `requestList`：好友请求列表(NSArray<IMFriendRequestInfoOC>)  

  `IMFriendRequestInfoOC`结构体成员如下：  
  `askerID`：NSString类型，请求方ID
  `askerNickname`：NSString类型，请求方昵称
  `inviteeID`：NSString类型，受邀方ID
  `inviteeNickname`：NSString类型，受邀方昵称
  `validateInfo`：NSString类型，验证信息
  `status`：枚举类型，好友请求状态，YIMAddFriendStatusOC:
                 YIMSTATUS_ADD_SUCCESS=0         //添加完成
                 YIMSTATUS_ADD_FAILED=1          //添加失败  
                 YIMSTATUS_WAIT_OTHER_VALIDATE=2 //等待对方验证 
                 YIMSTATUS_WAIT_ME_VALIDATE=3    //等待我验证
  `createTime`：unsigned int类型，请求的发送或接收时间               

## 全球布点选择 

- 接口：

  ``` obj-c
  +(void) SetServerZone:(YouMeIMServerZoneOC) zone;
  ```

- 参数说明：
  `zone`：选择合适的服务器区域，** 必须在SDK初始化之前设置 **。

- 返回：
  无。
  
## 服务器部署地区定义

``` obj-c
typedef enum
{
    YouMeIMServerZone_China = 0,        // 中国
    YouMeIMServerZone_Singapore = 1,    // 新加坡
    YouMeIMServerZone_America = 2,      // 美国
    YouMeIMServerZone_HongKong = 3,     // 香港
    YouMeIMServerZone_Korea = 4,        // 韩国
    YouMeIMServerZone_Australia = 5,    // 澳洲
    YouMeIMServerZone_Deutschland = 6,  // 德国
    YouMeIMServerZone_Brazil = 7,       // 巴西
    YouMeIMServerZone_India = 8,        // 印度
    YouMeIMServerZone_Japan = 9,        // 日本
    YouMeIMServerZone_Ireland = 10,     // 爱尔兰
    YouMeIMServerZone_Thailand = 11,    // 泰国
    YouMeIMServerZone = 12,	           // 台湾
    
    YouMeIMServerZone_Unknow = 9999
}YouMeIMServerZoneOC;
```  

## 错误码定义
| 错误码                                               | 含义         |
| -------------                                       | -------------|
| YouMeIMCode_Success = 0                        | 成功|
| YouMeIMCode_EngineNotInit = 1                  | IM SDK未初始化|
| YouMeIMCode_NotLogin = 2                       | IM SDK未登录|
| YouMeIMCode_ParamInvalid = 3                   | 无效的参数|
| YouMeIMCode_TimeOut = 4                        | 超时|
| YouMeIMCode_StatusError = 5                    | 状态错误|
| YouMeIMCode_SDKInvalid = 6                     | Appkey无效|
| YouMeIMCode_AlreadyLogin = 7                   | 已经登录|
| YouMeIMCode_LoginInvalid = 1001                | 登录无效|
| YouMeIMCode_ServerError = 8                    | 服务器错误|
| YouMeIMCode_NetError = 9                       | 网络错误|
| YouMeIMCode_LoginSessionError = 10             | 登录状态出错|
| YouMeIMCode_NotStartUp = 11                    | SDK未启动|
| YouMeIMCode_FileNotExist = 12                  | 文件不存在|
| YouMeIMCode_SendFileError = 13                 | 文件发送出错|
| YouMeIMCode_UploadFailed = 14                  | 文件上传失败,上传失败 一般都是网络限制上传了|
| YouMeIMCode_UsernamePasswordError = 15,          | 用户名密码错误|
| YouMeIMCode_UserStatusError = 16,                | 用户状态错误(无效用户)|
| YouMeIMCode_MessageTooLong = 17,                 | 消息太长|
| YouMeIMCode_ReceiverTooLong = 18,                | 接收方ID过长（检查房间名）|
| YouMeIMCode_InvalidChatType = 19,                | 无效聊天类型|
| YouMeIMCode_InvalidReceiver = 20,                | 无效用户ID|
| YouMeIMCode_UnknowError = 21,                    | 未知错误|
| YouMeIMCode_InvalidAppkey = 22,                  | AppKey无效|
| YouMeIMCode_ForbiddenSpeak = 23,                 | 被禁止发言|
| YouMeIMCode_CreateFileFailed = 24,               | 创建文件失败|
| YouMeIMCode_UnsupportFormat = 25                 | 不支持的文件格式|
| YouMeIMCode_ReceiverEmpty = 26                   | 接收方为空|
| YouMeIMCode_RoomIDTooLong = 27                   | 房间名太长|
| YouMeIMCode_ContentInvalid = 28                  | 聊天内容严重非法|
| YouMeIMCode_NoLocationAuthrize = 29              | 未打开定位权限|
| YouMeIMCode_UnknowLocation = 30                  | 未知位置|
| YouMeIMCode_Unsupport = 31                       | 不支持该接口|
| YouMeIMCode_NoAudioDevice = 32                   | 无音频设备|
| YouMeIMCode_AudioDriver = 33                     | 音频驱动问题|
| YouMeIMCode_DeviceStatusInvalid = 34             | 设备状态错误|
| YouMeIMCode_ResolveFileError = 35                | 文件解析错误|
| YouMeIMCode_ReadWriteFileError = 36              | 文件读写错误|
| YouMeIMCode_NoLangCode = 37                      | 语言编码错误|
| YouMeIMCode_TranslateUnable = 38                 | 翻译接口不可用|
| YIMErrorcode_SpeechAccentInvalid = 39,           | 语音识别方言无效|
| YIMErrorcode_SpeechLanguageInvalid = 40,         | 语音识别语言无效|
| YIMErrorcode_HasIllegalText = 41,                | 消息含非法字符|
| YIMErrorcode_AdvertisementMessage = 42,          | 消息涉嫌广告|
| YIMErrorcode_AlreadyBlock = 43,                  | 用户已经被屏蔽|
| YIMErrorcode_NotBlock = 44,                      | 用户未被屏蔽|
| YIMErrorcode_MessageBlocked = 45,                | 消息被屏蔽|
| YouMeIMCode_LocationTimeout = 46,                | 定位超时|
| YouMeIMCode_NotJoinRoom = 47,                    | 未加入该房间| 
| YouMeIMCode_LoginTokenInvalid = 48,              | 登录token错误|
| YouMeIMCode_CreateDirectoryFailed = 49,          | 创建目录失败|
| YouMeIMCode_InitFailed = 50,                     | 初始化失败|
| YouMeIMCode_Disconnect = 51,                     | 与服务器断开|
| YouMeIMCode_TheSameParam = 52,                   | 设置参数相同|
| YouMeIMCode_QueryUserInfoFail = 53,              | 查询用户信息失败|
| YouMeIMCode_SetUserInfoFail = 54,                | 设置用户信息失败|
| YouMeIMCode_UpdateUserOnlineStateFail = 55,      | 更新用户在线状态失败|
| YouMeIMCode_NickNameTooLong = 56,                | 昵称太长(> 64 bytes)|
| YouMeIMCode_SignatureTooLong = 57,               | 个性签名太长(> 120 bytes)|
| YouMeIMCode_NeedFriendVerify = 58,               | 需要好友验证信息|
| YouMeIMCode_BeRefuse = 59,	                      | 添加好友被拒绝|
| YouMeIMCode_HasNotRegisterUserInfo = 60,         | 未注册用户信息|
| YouMeIMCode_AlreadyFriend = 61,                 | 已经是好友|
| YouMeIMCode_NotFriend = 62,                     | 非好友|
| YouMeIMCode_NotBlack = 63,                      | 不在黑名单中|
| YouMeIMCode_PhotoUrlTooLong = 64,               | 头像url过长(>500 bytes)|
| YouMeIMCode_PhotoSizeTooLarge = 65,             | 头像太大(>100 kb)|
| YouMeIMCode_ChannelMemberOverflow = 66,         | 达到频道人数上限|
| YouMeIMCode_ALREADYFRIENDS = 1000,              | 已为好友|
| YouMeIMCode_LoginInvalid = 1001,                | 登录无效|
| YouMeIMCode_PTT_Start = 2000,                    | 开始录音|
| YouMeIMCode_PTT_Fail = 2001,                     | 录音失败|
| YouMeIMCode_PTT_DownloadFail = 2002,           | 语音消息文件下载失败|
| YouMeIMCode_PTT_GetUploadTokenFail = 2003,     | 获取语音消息Token失败|
| YouMeIMCode_PTT_UploadFail = 2004,             | 语音消息文件上传失败|
| YouMeIMCode_PTT_NotSpeech = 2005,              | 未检测到语音|
| YouMeIMCode_PTT_DeviceStatusError = 2006,      | 语音设备状态错误
| YouMeIMCode_PTT_IsSpeeching = 2007,            | 录音中|
| YouMeIMCode_PTT_FileNotExist = 2008,           | 文件不存在|
| YouMeIMCode_PTT_ReachMaxDuration = 2009,       | 达到最大时长限制|
| YouMeIMCode_PTT_SpeechTooShort = 2010,         | 录音时间太短|
| YouMeIMCode_PTT_StartAudioRecordFailed = 2011, | 启动录音失败|
| YouMeIMCode_PTT_SpeechTimeout = 2012,          | 音频输入超时|
| YouMeIMCode_PTT_IsPlaying = 2013,              | 正在播放|
| YouMeIMCode_PTT_NotStartPlay = 2014,           | 未开始播放|
| YouMeIMCode_PTT_CancelPlay = 2015,             | 主动取消播放|
| YouMeIMCode_PTT_NotStartRecord = 2016,         | 未开始语音|
| YouMeIMCode_PTT_NotInit = 2017,                | 未初始化|
| YouMeIMCode_PTT_InitFailed = 2018,             | 初始化失败|
| YouMeIMCode_PTT_Authorize = 2019,              | 录音权限|
| YouMeIMCode_PTT_StartRecordFailed = 2020,      | 启动录音失败|
| YouMeIMCode_PTT_StopRecordFailed = 2021,       | 停止录音失败|
| YouMeIMCode_PTT_UnsupprtFormat = 2022,         | 不支持的格式|
| YouMeIMCode_PTT_ResolveFileError = 2023,       | 解析文件错误|
| YouMeIMCode_PTT_ReadWriteFileError = 2024,     | 读写文件错误|
| YouMeIMCode_PTT_ConvertFileFailed = 2025,      | 文件转换失败|
| YouMeIMCode_PTT_NoAudioDevice = 2026,          | 无音频设备|
| YouMeIMCode_PTT_NoDriver = 2027,               | 驱动问题|
| YouMeIMCode_PTT_StartPlayFailed = 2028,        | 启动播放失败|
| YouMeIMCode_PTT_StopPlayFailed = 2029,         | 停止播放失败|
| YouMeIMCode_PTT_RecognizeFailed = 2030,        | 识别失败|
| YouMeIMCode_Fail = 10000                       | 语音服务启动失败|

### 备注：
实际Demo可以点击此处下载->[Youme Im Demo for iOS](http://dl2.youme.im/release/YIM_iOS2_2.x.zip)


