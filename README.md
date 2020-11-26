pragma solidity >=0.4.22 <0.6.0;

contract vdsGame {
    using SafeMath for uint256;
    
    /**
     *VDS共识合约共助参数 
     */
    address public owner;
	//间隔时间 3天
    uint256 public gameTime = 3 days;
	//初始金额 200
    uint256 public gameSettingNum = 200000000;
	//最多单数 10单
	uint256 public odd = 10;
	//收益比例 5%
	uint256 public earningsRatio = 50;


    //用户数量
	uint256 public users;
	
	/**
	 *映射 
	 */
	mapping (address => userValue[]) public userValues;
	mapping (address => uint256) public userNum;
	mapping (address => bool) public isReg;
	
	//用户结构体（金额，入金时间）
	struct userValue{
	    uint256 value;
	    uint256 timeOfUser;
	    bool isOut;
	}
	
	//限制合约拥有者操作
	modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }
    
	//限制入金金额 gameSettingNum的倍数
    modifier onlySetting(uint256 _value){
        bool flag = false;
        for(uint256 a=1;a<=odd;a++){
            if(_value==gameSettingNum.mul(a)){
                flag=true;
            }
        }
        require(flag);
        _;
    }

    /**
     *触发事件
     */
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    event joinGame(address indexed user,uint256 amount);
    event setting(uint256 time,uint256 num,uint256 odd,uint256 earningsRatio);
    event out(address indexed user,uint256 amount,uint256 time);
    
    /**
     *构造函数
     */
    constructor() public {
        owner = msg.sender;
    }
	
    //加入方法 参数_owner   30万地址前3%    30万地址后1.5%
    function join(address payable _owner) payable external onlySetting(msg.value) returns(bool){
		if(!isReg[msg.sender]){
			users = users.add(1);
			isReg[msg.sender]=true;
		}
      require(msg.value>=0,"The amount is not enough");
	  if(users>=300000){
		_owner.transfer((msg.value.mul(30)).div(1000));
	  }else{
		_owner.transfer((msg.value.mul(15)).div(1000));
	  }
      userValues[msg.sender].push(userValue(msg.value,now,false));
      userNum[msg.sender] = userNum[msg.sender].add(1);
      emit joinGame(msg.sender,msg.value);
      return true;
    }
  
    //提取金额的方法 _user 提取的用户地址，_num对应用户的入金数组下标
    function rollOut(address payable _user,uint8 _num) payable public onlyOwner returns(bool){
      require(_user!=address(0),"The address is null");
      require(_num>=0,"The amount is not enough");
      userValue memory user = userValues[_user][_num];
      require(!user.isOut,"Already extracted");
      uint256 userV = user.value;
      uint256 time = user.timeOfUser;
      uint256 amount = userV.add((userV.mul(earningsRatio)).div(1000));
      require((now-time)>=gameTime,"It's early");
      if(userV>0&&address(this).balance>=amount){
        _user.transfer(amount);
      }
      userValues[_user][_num].isOut=true;
      emit out(_user,amount,now);
      return true;
    }
  
    //合约参数 _time时间 _num金额 _odd单数 _earningsRatio收益比例
    function changeGameSetting(uint256 _time,uint256 _num,uint256 _odd,uint256 _earningsRatio) public onlyOwner{
      gameTime = _time;
      gameSettingNum = _num;
      odd=_odd;
      earningsRatio=_earningsRatio;
      emit setting(_time,_num,_odd,_earningsRatio);
    }
  
  
    //合约拥有者 newOwner新的合约拥有者
    function transferOwnership(address newOwner) public onlyOwner returns(bool){
      require(newOwner != address(0),"The address is null");
      owner = newOwner;
      emit OwnershipTransferred(owner, newOwner);
      return true;
    }
  
    //获取合约VDS余额
    function getBalance() public view returns(uint256){
      return address(this).balance;
    }
  
    //获取指定用户的入金次数 _owner 指定用户
    function getNumsByOwner(address _owner) external view returns(uint256) {
        return userNum[_owner];
    }
}
    
library SafeMath {
    function mul(uint256 a, uint256 b) internal pure returns (uint256) {if (a == 0) {return 0;} uint256 c = a * b; assert(c / a == b); return c;}
    function div(uint256 a, uint256 b) internal pure returns (uint256) {uint256 c = a / b; return c;}
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {assert(b <= a); return a - b;}
    function add(uint256 a, uint256 b) internal pure returns (uint256) {uint256 c = a + b; assert(c >= a); return c;}
}
