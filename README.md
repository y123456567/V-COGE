# VDS_hlep
VDS链上合约
pragma solidity >=0.4.22 <0.6.0;
contract vdsGame {
    using SafeMath for uint256;
    address public owner;
	//触发地址转出间隔时间
    uint256 public gameTime = 3 days;
	//初始金额
    uint256 public gameSettingNum = 200000000;
	uint256 public users;

	mapping (address => userValue[]) public userValues;
	mapping (address => uint256) public userNum;
	
	//用户结构体（金额，入金时间）
	struct userValue{
	    uint256 value;
	    uint256 timeOfUser;
	}
	
	//限制合约拥有者操作
	modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }
    
	//限制入金金额 gameSettingNum的倍数
    modifier onlySetting(uint256 _value){
        bool flag = false;
        for(uint256 a=1;a<=10;a++){
            if(_value==gameSettingNum.mul(a)){
                flag=true;
            }
        }
        require(flag);
        _;
    }

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    event joinGame(address indexed user,uint256 amount);
    event setting(uint256 time,uint256 num);
    event out(address indexed user,uint256 amount,uint256 time);
		
    constructor() public {
        owner = msg.sender;
    }
	
	//加入方法 参数_owner 转出地址 在30万地址钱5% 超过30万地址3% 
  function join(address payable _owner) payable external onlySetting(msg.value) returns(bool){
		if(userValues[msg.sender][0].value==0){
			users = users.add(1);
		}
      require(msg.value>=0,"The amount is not enough");
	  if(users>=300000){
		_owner.transfer((msg.value.mul(3)).div(100));
	  }else{
		_owner.transfer((msg.value.mul(5)).div(100));
	  }
      userValues[msg.sender].push(userValue(msg.value,now));
      userNum[msg.sender] = userNum[msg.sender].add(1);
      emit joinGame(msg.sender,msg.value);
      return true;
  }
  
  //提取金额的方法 _user 提取的用户地址，_num对应用户的入金数组下标
  function rollOut(address payable _user,uint8 _num) payable public onlyOwner returns(bool){
      require(_user!=address(0),"The address is null");
      require(_num>=0,"The amount is not enough");
      uint256 userV = userValues[_user][_num].value;
      uint256 time = userValues[_user][_num].timeOfUser;
      uint256 amount = userV.add((userV.mul(5)).div(100));
      require((now-time)>=gameTime,"It's early");
      require(userV>0&&address(this).balance>=amount,"balance is too low");
      _user.transfer(amount);
      emit out(_user,amount,now);
      return true;
  }
  
  //更改游戏参数 _time时间 _num金额
  function changeGameSetting(uint256 _time,uint256 _num) public onlyOwner{
      gameTime = _time;
      gameSettingNum = _num;
      emit setting(_time,_num);
  }
  
  
  //更改合约拥有者 newOwner新的合约拥有者
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
