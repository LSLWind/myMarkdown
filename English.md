dipatcher v. 调度，派遣

upload v. 上传

render v.给予，提供

velocity n.速度，速率

snippet n.片段

widget n.小器物，小玩意

sequence n.一系列，次序，序列;v.按顺序排列

radio n.收音机，无线电通信;v.用无线电发送



```
//包装器业务异常实现
public class BusinessException extends Exception implements CommonError{
    private CommonError commonError;
    //直接接受EmBusinessError的传参用于构造业务异常
    public BusinessException(CommonError commonError){
        super();
        this.commonError=commonError;
    }
    //接受自定义errMsg的方式构造业务异常
    public BusinessException(CommonError commonError, String errMsg){
        super();
        this.commonError=commonError;
        this.commonError.setErrMsg(errMsg);
    }
    ...//重写接口方法
}
```

