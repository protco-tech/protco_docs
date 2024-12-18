
# List of API endpoints related to mobile app

**TODO:** _Add missing endpoints, if any. Complete the API doc_  

## Chat Api  

```TEXT
{host}/business-app/v1/common/getMiffyConfig  
{host}/business-app/v1/file/uploadFile  
```

## Common Api

```TEXT
{host}/business-app/v1/mobile/settings
```

## Connect Api

```TEXT
{host}/business-app/v1/device/matter/addMatterShare
{host}/business-app/v1/device/bind/bindMatter
{host}/business-app/v1/device/bindToAsset
{host}/business-app/v1/device/bindVirtualDeviceToAsset
{host}/business-app/v1/device/bind/checkBindResult/{uuid}
{host}/business-app/v1/device/checkIsBind
{host}/business-app/v1/distributionNet/dataEncrypt
{host}/business-app/v1/networkGuide/getByAppProductTypeId
{host}/business-app/v1/networkGuide/getByAppProductTypeId
{host}/business-app/v1/short/link/getByShortCode/{shortType}/{shortCode}
{host}/business-app/v1/short/link/getByShortCode
{host}/business-app/v1/bluetooth/secret/getByDeviceUuid
{host}/business-app/v1/ipc/getToken/{assetId}
{host}/business-app/v1/device/matter/matterId
{host}/business-app/v1/product/getModel
{host}/business-app/v1/product/getNetworkGuide
{host}/business-app/v1/category/getCategoryListByPid/{pid}
{host}/business-app/v1/device/getSimpleDeviceInfo
{host}/business-app/v1/category/top
{host}/business-app/v1/device/matter/shareMatterPair
{host}/business-app/v1/device/updateDeviceVersion
```

## Device Api

```TEXT
{host}/business-app/v1/device/batchChangeAssetAffiliation
{host}/business-app/v1/ota/batchCheckUpgrade/{assetId}
{host}/business-app/v1/device/matter/checkIsShare
{host}/business-app/v1/ota/checkUpgrade/{deviceId}/{firmwareType}
{host}/business-app/v1/asset/delete/{assetId}
{host}/business-app/v1/agora/getAgoraToken
{host}/business-app/v1/asset/detail/{assetId}
{host}/business-app/v1/nfc/getBindRefsByUUIDs
{host}/business-app/v1/city/list
{host}/business-app/v1/device/getGroupAndScene/{deviceId}
{host}/business-app/v1/device/getByDeviceId/{deviceId}
{host}/business-app/v1/device/getByUUID/{uuid}
{host}/business-app/v1/device/dp/getDeviceShadow
{host}/business-app/v1/device/getDpInfos/{deviceId}
{host}/business-app/v1/device/dp/group/{groupId}
{host}/business-app/v1/device/dp/getGroupShadow
{host}/business-app/v1/device/getHomeDeviceAndGroupList
{host}/business-app/v1/product/getPanel/{productId}/{deviceType}
{host}/business-app/v1/product/{productId}/getLang
{host}/business-app/v1/device/getReplaceGatewayList/{rootAssetId}/{pid}
{host}/business-app/v1/device/getReplaceListByGatewayId
{host}/business-app/v1/product/getRnNetworkGuideByProductId/{productId}/{deviceType}
{host}/business-app/v1/networkGuide/getByProductId
{host}/business-app/v1/device/initDevice
{host}/business-app/v1/device/lowpowerWakeupByUUID/{uuid}
{host}/business-app/v1/device/command/notifyDeviceJoin
{host}/business-app/v1/device/replaceGateway
{host}/business-app/v1/device/matter/reportMatterPair
{host}/business-app/v1/device/command/configSettings
{host}/business-app/v1/device/setHomeResultSort
{host}/business-app/v1/ota/startUpgrade
{host}/business-app/v1/ota/submitUpgrade
{host}/business-app/v1/device/unbindFromAsset
{host}/business-app/v1/device/updateRemindStatus
```

## Group Device Api

```TEXT
{host}/business-app/v1/deviceGroup/detail/{groupId}
{host}/business-app/v1/device/getGroupDeviceList
{host}/business-app/v1/deviceGroup/getScene/{groupId}
{host}/business-app/v1/deviceGroup/del/{groupId}
{host}/business-app/v1/deviceGroup/saveOrUpdate
{host}/business-app/v1/deviceGroup/syncGateway
```

## Device Share Api

```TEXT
{host}/business-app/v1/deviceShare/addShare
{host}/business-app/v1/device/batchChangeAssetAffiliation
{host}/business-app/v1/asset/delete/{assetId}
{host}/business-app/v1/deviceShare/deleteShare/{id}
{host}/business-app/v1/deviceShare/remark
{host}/business-app/v1/deviceShare/getByDeviceIdAndTargetUserId/{deviceId}/{targetUserId}
{host}/business-app/v1/deviceShare/getByRefAndTargetUserId/{refType}/{refId}/{targetUserId}
{host}/business-app/v1/deviceShare/getShareByShareUserId/{shareUserId}
{host}/business-app/v1/deviceShare/getShareUserList
{host}/business-app/v1/deviceShare/getTargetUserByDeviceId/{deviceId}
{host}/business-app/v1/deviceShare/getTargetUserByRef/{refType}/{refId}
{host}/business-app/v1/device/initDevice
{host}/business-app/v1/device/lowpowerWakeupByUUID/{uuid}
{host}/business-app/v1/deviceGroup/del/{groupId}
{host}/business-app/v1/ota/startUpgrade
{host}/business-app/v1/ota/submitUpgrade
{host}/business-app/v1/device/unbindFromAsset
{host}/business-app/v1/device/updateRemindStatus
```

## Home Api

```TEXT
{host}/business-app/v1/asset/memberInviteConfirm
{host}/business-app/v1/device/batchChangeAssetAffiliation
{host}/business-app/v1/asset/create
{host}/business-app/v1/asset/create
{host}/business-app/v1/asset/delete/{assetId}
{host}/business-app/v1/asset/{assetId}/memberExit
{host}/business-app/v1/asset/detail/{assetId}
{host}/business-app/v1/asset/assetTree
{host}/business-app/v1/device/getUserDeviceList
{host}/business-app/v1/device/getHomeDeviceAndGroupList
{host}/business-app/v1/scene/getHomeOneKeyList
{host}/business-app/v1/asset/myInviteList
{host}/business-app/v1/asset/{assetId}/inviteList
{host}/business-app/v1/asset/memberInvite
{host}/business-app/v1/scene/onkeyExcute/{sceneId}
{host}/business-app/v1/deviceGroup/del/{groupId}
{host}/business-app/v1/asset/delete/{assetId}/member/{memberId}
{host}/business-app/v1/asset/{assetId}/selected
{host}/business-app/v1/asset/setMemberRole
{host}/business-app/v1/asset/setSorts
{host}/business-app/v1/asset/update
```

## Feedback Api

```TEXT
{host}/business-app/v1/problem/feedbackCategoryList
{host}/business-app/v1/problem/suggestCategoryList
{host}/business-app/v1/file/uploadFile @Multipart
```

## Mqtt Api

```TEXT
{host}/business-app/v1/device/command/checkSignal
{host}/business-app/v1/device/command/batchPropsIssue
{host}/business-app/v1/device/command/groupPropsIssue
{host}/business-app/v1/device/command/onekeyExecuteIssue
{host}/business-app/v1/device/command/otaIssue
{host}/business-app/v1/device/command/propsIssue
{host}/business-app/v1/device/command/propsReport
{host}/business-app/v1/device/command/scanIssue
{host}/business-app/v1/device/command/subAdd
{host}/business-app/v1/device/command/otaBatchIssue
{host}/business-app/v1/device/command/subReplace
```

## Msg Api

```TEXT
{host}/business-app/v1/scene/list
{host}/business-app/v1/asset/detail/{assetId}
{host}/business-app/v1/asset/assetTree
{host}/business-app/v1/device/getHomeDeviceAndGroupList
{host}/business-app/v2/message/page
{host}/business-app/v1/my/getSettings
{host}/business-app/v2/message/msgGroupList
{host}/business-app/v2/message/read
{host}/business-app/v2/message/remove
{host}/business-app/v1/my/saveSettings

{host}/business-app/v2/message/removeByQuery
{host}/business-app/v2/message/remove
{host}/business-app/v1/message/list
{host}/business-app/v1/my/getSettings
{host}/business-app/v1/my/saveSettings
```

## RN Panel Api

```TEXT
{host}/business-app/v1/device/getDpInfos/{deviceId}
{host}/business-app/v1/device/dp/group/{groupId}
{host}/business-app/v1/rn/getAppPanel/{panelCode}/{deviceType}
{host}/business-app/v1/device/getListByGatewayId/{gatewayId}
{host}/business-app/v1/product/{productId}/getPanel
{host}/business-app/v1/product/getPanel/{productId}/{deviceType}
{host}/business-app/v1/product/{productId}/getLang
{host}/business-app/v1/device/lowpowerWakeupByUUID/{uuid}
```

## NFC Api

```TEXT
{host}/business-app/v1/scene/list
{host}/business-app/v1/asset/detail/{assetId}
{host}/business-app/v1/asset/assetTree
{host}/business-app/v1/nfc/getBindRefsByUUIDs
{host}/business-app/v1/device/getHomeDeviceAndGroupList
{host}/business-app/v1/nfc/bind
{host}/business-app/v1/nfc/{uuid}/unbind
{host}/business-app/v1/nfc/updateName
```

## Scene Api

```TEXT
{host}/business-app/v1/scene/changeEnabeldStatus
{host}/business-app/v1/scene/create
{host}/business-app/v1/scene/delete/{sceneId}
{host}/business-app/v1/stream/rule/user/instance/executeInstance
{host}/business-app/v1/scene/list
{host}/business-app/v1/stream/rule/user/instance/template/getAppRecommendList
{host}/business-app/v1/nfc/getBindRefsByUUIDs
{host}/business-app/v1/stream/rule/user/instance/getByInstanceId
{host}/business-app/v1/stream/rule/user/instance/getInstanceList
{host}/business-app/v1/scene/getInfo/{sceneId}
{host}/business-app/v1/device/getSceneDevicesDpShadow
{host}/business-app/v1/scene/icons
{host}/business-app/v1/device/getSceneSupportDpList/{deviceId}
{host}/business-app/v1/device/getSupportSceneDeviceList
{host}/business-app/v1/stream/rule/user/instance/template/incrUseNum
{host}/business-app/v1/stream/rule/user/instance/syncGateway
{host}/business-app/v1/scene/onkeyExcute/{sceneId}
{host}/business-app/v1/stream/rule/user/instance/queryInstanceLogPage
{host}/business-app/v1/system/icon/queryList
{host}/business-app/v1/stream/rule/user/instance/removeByInstanceId
{host}/business-app/v1/stream/rule/user/instance/add
{host}/business-app/v1/stream/rule/user/instance/updateByInstanceId
{host}/business-app/v1/scene/update
{host}/business-app/v1/stream/rule/user/instance/updateSort
{host}/business-app/v1/stream/rule/user/instance/updateStatus
```

## My Page Api

```TEXT
{host}/business-app/v1/version/checkUpdate
{host}/business-app/v1/user/dev/getUserDevList
{host}/business-app/v1/user/dev/unbindUserDev
{host}/business-app/v1/user/updateInfo
{host}/business-app/v1/file/uploadFile
```

## User Api

```TEXT
{host}/business-app/v1/user/dev/bindUserDev
{host}/business-app/v1/user/cancelAccount
{host}/business-app/v1/verifyCode/checkVerifyCode
{host}/business-app/v1/user/destroy/destroyUser
{host}/business-app/v2/user/password/find/sendFindPasswordCode
{host}/business-app/v1/common/getAppCommonConfig
{host}/business-app/v1/common/getAppMobilePushConfig
{host}/business-app/v1/common/getCaptchaCode
{host}/business-app/v1/country/getInfo
{host}/business-app/v1/common/getCountryList
{host}/business-app/v1/common/getCurrentCountry
{host}/business-app/v1/common/getDataCenterList
{host}/business-app/v1/enums/getRegisterType
{host}/business-app/v1/country/getTimezoneList
{host}/business-app/v1/user/profile
{host}/business-app/v1/user/registry
{host}/business-app/v1/user/register/registryByUserName
{host}/business-app/v1/user/resetPassword
{host}/business-app/v1/user/password/find/resetPassword
{host}/business-app/v2/user/bind/sendBindCode
{host}/business-app/v1/user/dev/sendBindUserDevCode
{host}/business-app/v2/user/dev/sendBindUserDevCode
{host}/business-app/v2/user/destroy/sendDestroyCode
{host}/business-app/v1/user/password/find/sendFindPasswordCode
{host}/business-app/v1/user/register/sendRegisterCode
{host}/business-app/v2/user/register/sendRegisterCode
{host}/business-app/v1/common/sendResetMail
{host}/business-app/v2/user/bind/sendUnbindCode
{host}/business-app/v1/user/password/update/sendUpdatePasswordCode
{host}/business-app/v1/verifyCode/send
{host}/business-app/v1/user/password/update/updatePassword
{host}/business-app/v2/user/password/update/sendUpdatePasswordCode
```

## Voice Api

```TEXT
{host}/business-app/v1/google/skill/createOauth2Code
{host}/business-app/v1/my/getTrdRadioList
{host}/business-app/v1/google/skill/isBind
{host}/business-app/v1/alexa/skill/disableSkill
{host}/business-app/v1/alexa/skill/enableSkill
{host}/business-app/v1/alexa/skill/isBind
```
