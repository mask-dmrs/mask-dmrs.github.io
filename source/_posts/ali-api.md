---
title: ali-api-util
date: 2017-12-06 23:45:02
permalink: ali-api-util
categories: Gists
tags:
 - java
---

```java
@Component
@NotThreadSafe
public class AliApiClientUtil implements InitializingBean {

    @Value("${aliapi.appKey:no_config}")
    private String appKey;

    @Value("${aliapi.appSecret:no_config}")
    private String appSecret;

    @Value("${aliapi.enable:false}")
    private boolean enable;

    private AliApiClient asyncClient;

    @Override
    public void afterPropertiesSet() throws Exception {
        if (enable && (Constant.DEF_CONFIG.equals(appKey) || Constant.DEF_CONFIG.equals(appSecret))) {
            throw new RuntimeException(String.format("阿里云第三方API初始化,缺失参数appKey:{0},appSecret{1}", new Object[]{appKey, appSecret}));
        }
        asyncClient = AliApiClient.newBuilder()
                .appKey(appKey)
                .appSecret(appSecret)
                .build();
    }

    public static class Builder extends BaseApiClientBuilder<Builder, AliApiClient> {

        @Override
        protected AliApiClient build(BuilderParams params) {
            return new AliApiClient(params);
        }
    }

    public static class AliApiClient extends BaseApiClient {

        private AliApiClient(BuilderParams builderParams) {
            super(builderParams);
        }

        public static Builder newBuilder() {
            return new Builder();
        }

        public static AliApiClient getInstance() {
            return getApiClassInstance(AliApiClient.class);
        }

        public void asyncInvoke0(ApiRequest apiRequest, ApiCallBack callBack) {
            asyncInvoke(apiRequest, callBack);
        }

        public ApiResponse syncInvoke0(ApiRequest apiRequest) {
            return syncInvoke(apiRequest);
        }
    }

    /**
     *{
     *"resp":{
     *  "code":0,
     *  "desc":"OK"
     *  } ,
     *  "data":{},
     * }
     * 银行卡二、三、四元素实名认证
     * https://market.aliyun.com/products/57000002/cmapi014429.html?spm=5176.2020520132.101.5.2lTTNI#sku=yuncode842900008
     * @see AliHttpStatusEm
     * @see com.dotnar.usc.common.util.aliapi.bankcardverifi.BankCardVerifiStatusEm
     * @param bankCardVerifiVo
     * @return
     */
    public ApiResponse syncBankCardVerifi(BankCardVerifiVo bankCardVerifiVo) {
        ApiRequest apiRequest = new ApiRequest(Scheme.HTTP, Method.GET, Constant.BANK_CARD_VERIFI_HOST, Constant.BANK_CARD_VERIFI_API_PATH);
        apiRequest.addParam("acct_name", bankCardVerifiVo.getAcctName(), ParamPosition.QUERY, false);
        apiRequest.addParam("acct_pan", bankCardVerifiVo.getAcctPan(), ParamPosition.QUERY, true);
        apiRequest.addParam("cert_id", bankCardVerifiVo.getCertId(), ParamPosition.QUERY, false);
        apiRequest.addParam("phone_num", bankCardVerifiVo.getPhoneNum(), ParamPosition.QUERY, false);
        return asyncClient.syncInvoke0(apiRequest);
    }

    /**
     * {
     * "resp": {
     *      "code": 0,
     *      "desc": "匹配"
     * },
     * "data": {
     *    "photo": "/9j/4AAQSkZJRgABAgAAAQAzb0a9jtkMbHG/o3+19P89K28x...",
     *    "address": "四川省雅安地区荥经县",
     *    "sex": "M",
     *    "birthday": "1989-9-28"
     *   }
     * }
     *
     * 身份证实名认证--返照片
     * https://market.aliyun.com/products/57000002/cmapi015194.html?spm=5176.2020520132.101.10.2lTTNI#sku=yuncode919400000
     * @param idPhotoVo
     * @see AliHttpStatusEm
     * @see com.dotnar.usc.common.util.aliapi.idphoto.IdPhotoStatusEm
     * @return
     */
    public ApiResponse syncIdphoto(IdPhotoVo idPhotoVo) {
        ApiRequest apiRequest = new ApiRequest(Scheme.HTTP, Method.GET, Constant.ID_PHOTO_HOST, Constant.ID_PHOTO_API_PATH);
        apiRequest.addParam("cardno", idPhotoVo.getCardno(), ParamPosition.QUERY, true);
        apiRequest.addParam("name", idPhotoVo.getName(), ParamPosition.QUERY, true);
        return asyncClient.syncInvoke0(apiRequest);
    }

}
```
 
常见的错误
```java
/**
 * @Date 2017/12/5 16:42
 * <p>
 * https://help.aliyun.com/document_detail/43906.html
 */

public enum AliHttpStatusEm {

    OK("OK", 200, "成功"),

    THROTTLED_BY_USER_FLOW_CONTROL("Throttled by USER Flow Control", 403, "因用户流控被限制"),

    THROTTLED_BY_APP_FLOW_CONTROL("Throttled by APP Flow Control", 403, "因APP流控被限制"),

    THROTTLED_BY_API_FLOW_CONTROL("Throttled by API Flow Control", 403, "因 API 流控被限制"),

    THROTTLED_BY_DOMAIN_FLOW_CONTROL("Throttled by DOMAIN Flow Control", 403, "因二级域名流控被限制，或因分组流控被限制"),

    QUOTA_EXHAUSTED("Quota Exhausted", 403, "调用次数已用完"),

    QUOTA_EXPIRED("Quota Expired", 403, "购买次数已过期"),

    USER_ARREARS("User Arrears", 403, "用户已欠费"),

    INVALID_ACCESS_KEY_ID_NOT_FOUND("InvalidAccessKeyId.NotFound", 404, "AccessKeyId 找不到"),

    INVALID_ACCESS_KEY_ID_INACTIVE("InvalidAccessKeyId.Inactive", 400, "AccessKeyId 被禁用。"),

    INVALID_TIME_STAMP_FORMAT("InvalidTimeStamp.Format", 400, "时间戳格式不对( Date 和 Timestamp)。"),

    INVALID_TIME_STAMP_EXPIRED("InvalidTimeStamp.Expired", 400, "用户时间和服务器时间超过15分钟。"),

    INVALID_SIGNATURE_METHOD("InvalidSignatureMethod", 400, "签名方法不支持。"),

    SIGNATURE_DOES_NOT_MATCH("SignatureDoesNotMatch", 400, "签名不通过。"),

    THROTTLING_USER("Throttling.User", 400, "用户调用频率超限。"),

    THROTTLING_API("Throttling.API", 400, "API 访问频率超限。");

    /**
     * 错误码
     */
    private String code;
    /**
     * http 状态码
     */
    private int statusCode;
    /**
     * 中文描述
     */
    private String desc;

    private final static Map<String, AliHttpStatusEm> ID_MAP = Arrays.stream(AliHttpStatusEm.values())
            .collect(Collectors.toMap(AliHttpStatusEm::getCode, Function.identity()));

    AliHttpStatusEm(String code, int statusCode, String desc) {
        this.code = code;
        this.statusCode = statusCode;
        this.desc = desc;
    }

    public String getCode() {
        return code;
    }

    public int getStatusCode() {
        return statusCode;
    }

    public String getDesc() {
        return desc;
    }

    public static AliHttpStatusEm getAliHttpStatusEmByStatusCode(String code) {
        return ID_MAP.get(code);
    }
}

```

银行卡验证错误状态
```java
/**
 * 银行卡验证状态码
 *
 * @author 
 * @Date 2017/12/5 17:31
 * <p>
 * https://market.aliyun.com/products/57000002/cmapi014429.html?spm=5176.2020520132.101.5.2lTTNI#sku=yuncode842900008
 */

public enum BankCardVerifiStatusEm {

    OK((short) 0, "OK"),

    CARD_IS_FORFEITED((short) 4, "此卡被没收，请于发卡方联系"),

    CARDHOLDER_AUTHENTICATION_FAILED((short) 5, "持卡人认证失败"),

    INVALID_CARD_NUMBER((short) 14, "无效卡号"),

    CARD_DOES_NOT_CORRESPOND_ISSUER((short) 15, "此卡无对应发卡方"),

    CARD_IS_NOT_INITIALIZED_OR_SLEEPS((short) 21, "该卡未初始化或睡眠卡"),

    CHEAT_TUNKA((short) 34, "作弊卡，吞卡"),

    ISSUER_DOES_NOT_SUPPORT_TRANSACTION((short) 40, "发卡方不支持的交易"),

    CARD_HAS__REPORTED((short) 41, "此卡已经挂失"),

    CARD_IS_FORFEITED_2((short) 43, "此卡被没收"),

    CARD_HAS_EXPIRED((short) 54, "该卡已过期"),

    CARD_ISSUER_DOES_NOT_ALLOW_THIS_TRANSACTION((short) 57, "发卡方不允许此交易"),

    RESTRICTED_CARD((short) 62, "受限制的卡"),

    WRONG_NUMBER_OF_PASSWORDS((short) 75, "密码错误次数超限"),

    FAILED((short) 96, "交易失败，请稍后重试");

    private short code;

    private String desc;

    private final static Map<Short, BankCardVerifiStatusEm> ID_MAP = Arrays.stream(BankCardVerifiStatusEm.values())
            .collect(Collectors.toMap(BankCardVerifiStatusEm::getCode, Function.identity()));

    BankCardVerifiStatusEm(short code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    public short getCode() {
        return code;
    }

    public String getDesc() {
        return desc;
    }

    public static BankCardVerifiStatusEm getBankCardVerifiStatusEmByCode(short code) {
        return ID_MAP.get(code);
    }
}
```
 
获取 身份证照片
```java
/**
 * @author 
 * @Date 2017/12/5 19:05
 * https://market.aliyun.com/products/57000002/cmapi015194.html?spm=5176.2020520132.101.10.2lTTNI#sku=yuncode919400000
 */

public enum IdPhotoStatusEm {

    OK((short) 0, "匹配"),

    MISMATCH((short) 5, "不匹配"),

    NOT_EXIST((short) 14, "无此身份证号码"),

    FAILED((short) 96, "交易失败，请稍后重试");

    private short code;

    private String desc;

    private final static Map<Short, IdPhotoStatusEm> ID_MAP = Arrays.stream(IdPhotoStatusEm.values())
            .collect(Collectors.toMap(IdPhotoStatusEm::getCode, Function.identity()));

    IdPhotoStatusEm(short code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    public short getCode() {
        return code;
    }

    public String getDesc() {
        return desc;
    }

    public static IdPhotoStatusEm getIdPhotoStatusEmByCode(short code) {
        return ID_MAP.get(code);
    }
}

```


