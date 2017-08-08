# Windows(PC) SDK 开发手册## 前言### 版本更新[传输门](http://dev.netease.im/docs/product/IM%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AF/%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97/Windows%E7%AB%AF%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97)### 开发集成[传输门](http://dev.netease.im/docs/product/IM%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AF/SDK%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90/Windows%E5%BC%80%E5%8F%91%E9%9B%86%E6%88%90)### 接口文档[传输门](http://dev.netease.im/docs/interface/%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AFWindows%E7%AB%AF/NIMSDKAPI_C/html/index.html)## 网易云信服务概要云信以提供客户端SDK（覆盖Android、iOS、Web、PC）和服务端OPEN API 的形式提供即时通讯云服务。开发者只需想好APP 的创意及UI 展现方案，根据APP 业务需求选择云信的相应功能接入即可。注：SDK 兼容的系统有 Windows xp(sp2及以上）、Windows 7、Windows 8/8.1、Windows 10。**SDK 从V3.2.5版本开始全面支持32位和64位程序接入。**## 开发准备### SDK内容* x64_dlls：存放64位Dll的文件夹* x86_dlls：存放32位Dll的文件夹* nim.dll： SDK核心功能模块；放在用户程序目录下* nim_audio.dll： 负责语音录制和播放；放在用户程序目录下* nim\_tools\_http.dll： 负责通用的http传输；nim.dll加载了此模块，需要放在用户程序目录下* nrtc.dll： 负责视频聊天功能；放在用户程序目录下* nrtc\_audio\_process.dll： 负责音频处理；放在用户程序目录下* nim\_audio\_hook.dll： 负责辅助采集播放器音频，由nrtc.dll调用；放在用户程序目录下，x64位暂时不提供该Dll。* nim_conf： SDK版本相关；放在用户程序目录下* nim\_c\_sdk： IM C 接口说明以及SDK所有定义的头文件（*_def.h头文件用户自己决定放在合适位置）。* nim\_chatroom\_c\_sdk: 聊天室 C 接口说明以及SDK所有定义的头文件（*_def.h头文件用户自己决定放在合适位置）。* nim\_sdk\_cpp\_vs2010：IM C接口的C++封装层，包括工程文件，工程文件为VS2010创建，用户添加进自己的工程后可根据开发环境升级该工程文件。* nim\_chatroom\_cpp\_vs2010：聊天室C接口的C++封装层，包括工程文件，工程文件为VS2010创建，用户添加进自己的工程后可根据开发环境升级该工程文件。* nim\_tools\_c\_sdk：SDK工具类（语音录制播放和Http）C接口说明以及所有定义的头文件（*_def.h头文件用户自己决定放在合适位置）。* nim\_tools\_cpp\_sdk：SDK工具类（语音录制播放和Http）C接口的C++封装层。SDK不提供debug版本的动态链接库供开发者调试，如果遇到问题请联系技术支持或在线客服。### 快速接入SDK关于如何快速集成SDK到现有项目，初始化SDK登录退出，发送/接收第一条IM消息，发送/接收第一条聊天室消息，如何开始基于[IM Demo源码](http://netease.im/im-sdk-demo)开发的信息请前往[Windows(PC) SDK Getting Started](http://dev.netease.im/docs/product/通用/新手接入/即时通讯/WindowsGettingStarted "target=_blank")### SDK数据目录当收到多媒体消息后，SDK 会负责下载这些多媒体文件，同时SDK 还要记录一些log，因此SDK 需要一个数据缓存目录。该目录由第三方App 通过nim\_client\_init 初始化接口传入，默认为存放到系统的AppData 目录下，App 传入一个目录名即可，SDK 会自动生成用户数据缓存目录。数据缓存目录默认为"{系统的AppData 目录}\\{App 传入的目录名}\\NIM\\{某个用户对应的用户数据目录}”，还可以由App 完全自定义用户数据目录，需要传入完整的路径，并确保读写权限正确。如果第三方App 需要清除缓存功能，可扫描该目录下的文件，按照你们的规则清理即可。具体某个用户对应的缓存目录下面包含如下子目录：- image：图片消息文件- audio：语音消息文件- res：其他资源文件SDK 提供了接口nim\_tool\_get\_user\_specific\_appdata\_dir 获取某个用户对应的具体类型的App data 目录（如图片消息文件存放目录，语音消息文件存放目录等）（注意：需要调用nim\_free\_buf 接口释放其返回的内存）。### 接口约定SDK 提供的所有API 都是 **C接口** ，根据模块划分为不同的API 文件，根据业务需要可以显式或者隐式的调用（3.4.0开始）。显式调用下，开发者可以在程序进行到任何时间点按需加载SDK dll， 功能完成后如果不需要继续使用NIM可以卸载SDK dll， 更加灵活和节省内存， 该模式下开发者只需要将SDK包里nim\_c\_sdk\\include 目录下的模块定义头文件（命名方式如nim\_xxx\_def.h）加入到工程项目中。隐式调用下，开发者除了链接lib文件外，需要将SDK包里nim\_c\_sdk 目录里的API 头文件（命名方式如nim\_xxx.h）以及相关常量定义的头文件等其他头文件一并加入到工程项目中。一般来说，每个模块都有对应的API 头文件和相关常量的定义头文件，命名上一一对应。SDK 提供了3种类型的接口。1. 全局广播类通知的注册接口。		常见的几类消息或通知是通过该注册的回调函数中通知开发者的，比如收到的消息，会话数据更新，用户相关数据的变更等，开发者需要在登录前提前注册好这些接口，例如：	C++		//注册数据同步结果的回调		nim::DataSync::RegCompleteCb(nbase::Bind(&nim_comp::DataSyncCallback::SyncCallback, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3));			/* 以下注册的回调函数，都是在收到服务器推送的消息或事件时执行的。因此需要在程序开始时就注册好。 */		//注册重连、被踢、掉线、多点登录、把移动端踢下线的回调		nim::Client::RegReloginCb(&nim_comp::LoginCallback::OnReLoginCallback);		nim::Client::RegKickoutCb(&nim_comp::LoginCallback::OnKickoutCallback);		nim::Client::RegDisconnectCb(&nim_comp::LoginCallback::OnDisconnectCallback);		nim::Client::RegMultispotLoginCb(&nim_comp::LoginCallback::OnMultispotLoginCallback);		nim::Client::RegKickOtherClientCb(&nim_comp::LoginCallback::OnKickoutOtherClientCallback);		nim::Client::RegSyncMultiportPushConfigCb(&nim_comp::MultiportPushCallback::OnMultiportPushConfigChange);			//注册返回发送消息结果的回调，和收到消息的回调。		nim::Talk::RegSendMsgCb(nbase::Bind(&nim_comp::TalkCallback::OnSendMsgCallback, std::placeholders::_1));		nim::Talk::RegReceiveCb(nbase::Bind(&nim_comp::TalkCallback::OnReceiveMsgCallback, std::placeholders::_1));		nim::Talk::RegRecallMsgsCallback(nbase::Bind(&nim_comp::TalkCallback::OnReceiveRecallMsgCallback, std::placeholders::_1, std::placeholders::_2));		nim::Talk::RegReceiveMessagesCb(nbase::Bind(&nim_comp::TalkCallback::OnReceiveMsgsCallback, std::placeholders::_1));		nim::MsgLog::RegMessageStatusChangedCb(nbase::Bind(&nim_comp::TalkCallback::OnMsgStatusChangedCallback, std::placeholders::_1));	C#		NIM.TalkAPI.OnReceiveMessageHandler += ReceiveMessageHandler;        NIM.TalkAPI.OnSendMessageCompleted += SendMessageResultHandler;		NIM.ClientAPI.RegMultiSpotLoginNotifyCb(OnMultiSpotLogin);        NIM.ClientAPI.RegKickOtherClientCb(OnKickOtherClient);		NIM.TalkAPI.OnReceiveMessageHandler += OnReceiveMessage;        NIM.TalkAPI.RegRecallMessageCallback(OnRecallMessage);	C		// 数据同步结果通知(nim_data_sync)		void nim_data_sync_reg_complete_cb(nim_data_sync_cb_func cb, const void *user_data);				// 接收会话消息通知(nim_talk)		void nim_talk_reg_receive_cb(const char *json_extension, nim_talk_receive_cb_func cb, const void *user_data);				// 接收系统消息通知(nim_sysmsg)		void nim_sysmsg_reg_sysmsg_cb(const char *json_extension, nim_sysmsg_receive_cb_func cb, const void *user_data);			// 发送消息结果通知(nim_talk)		void nim_talk_reg_arc_cb(const char *json_extension, nim_talk_arc_cb_func cb, const void *user_data);		// 发送自定义系统通知的结果通知(nim_talk)		void nim_talk_reg_custom_sysmsg_arc_cb(const char *json_extension, nim_custom_sysmsg_arc_cb_func cb, const void *user_data);		// 群组事件通知(nim_team)		void nim_team_reg_team_event_cb(const char *json_extension, nim_team_event_cb_func cb, const void *user_data);			// 帐号被踢通知(nim_client)		void nim_client_reg_kickout_cb(const char *json_extension, nim_json_transport_cb_func cb, const void *user_data);				// 网络连接断开通知(nim_client)		void nim_client_reg_disconnect_cb(const char *json_extension, nim_json_transport_cb_func cb, const void *user_data);			// 将本帐号的其他端踢下线的结果通知(nim_client)		void nim_client_reg_kickout_other_client_cb(const char *json_extension, nim_json_transport_cb_func cb, const void *user_data); 		// 好友通知 		void nim_friend_reg_changed_cb(const char *json_extension, nim_friend_change_cb_func cb, const void *user_data);			// 用户特殊关系通知		void nim_user_reg_special_relationship_changed_cb(const char *json_extension, nim_user_special_relationship_change_cb_func cb, const void *user_data);			...2. 异步接口。	回调函数作为参数，传入执行接口，然后执行接口时，会触发传入的回调函数。注意，回调函数中如果涉及到跨线程资源的需要开发者切换线程操作，即使不涉及到资源问题，也要保证回调函数中不会处理耗时任务，以免堵塞SDK底层线程。3. 同步接口。	为方便开发者使用，我们提供了一些同步接口，调用接口获取的内容同步返回，以block命名的都是同步接口，该类会堵塞SDK线程，谨慎使用。**从版本2.7.0开始，服务器和客户端上线了频控策略，与服务器有交互的接口(接口命名结尾为_online的查询接口以及命令类接口，如同意群邀请等)增加频控控制，开发者关注错误码416。**### C++ SDK为了方便桌面应用开发者更快的接入云信SDK，PC SDK下载包中还提供了官方的[C++ 封装层 项目文件及源码](https://github.com/netease-im/NIM_PC_SDK-CPP- "target=_blank")，接入和使用方法请看[Windows(PC) SDK开发手册(C++ 封装层)](http://dev.netease.im/docs/product/通用/Demo源码导读/PC通用/C++封装层 "target=_blank")，目前IM Demo(C++)源码就是通过接入和调用该SDK完成IM功能的。### C# SDK云信SDK还提供了C# 程序集，方便.net 开发人员接入，PC SDK下载包中包括官方的[C# 封装层 项目文件及源码](https://github.com/netease-im/NIM_PC_SDK-CSharp "target=_blank")，接入和使用方法请看[Windows(PC) SDK开发手册(C# 封装层)](http://dev.netease.im/docs/product/通用/Demo源码导读/PC通用/CSharp封装层 "target=_blank")，目前IM Demo(C#)源码就是通过接入和调用该SDK完成IM功能的。**如果开发者在调用C接口过程中或者解析接口返回结果过程中出现疑问，可以参考和借鉴C++封装层。**