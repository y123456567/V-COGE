pragma solidity >=0.4.22 <0.6.0;

contract vdsGame {
    using SafeMath for uint256;
    
    /**
     * 
     */
    address public owner;
 
    uint256 public gameTime = 3 days;
	 
    uint256 public gameSettingNum = 200000000;
 
	uint256 public odd = 10;
 
	uint256 public earningsRatio = 90;
 
	address payable public gasAddress;


    //用户数量
	uint256 public users;
	
	/**
	 *映射 
	 */
	mapping (address => userValue[]) public userValues;
	mapping (address => uint256) public userNum;
	mapping (address => bool) public isReg;
	

	struct userValue{
	    uint256 value;
	    uint256 timeOfUser;
	    bool isOut;
	}
	
 
	modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }
    
 
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
   
     */
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    event joinGame(address indexed user,uint256 amount);
    event setting(uint256 time,uint256 num,uint256 odd,uint256 earningsRatio,address indexed gasAddress);
    event out(address indexed user,uint256 amount,uint256 time);
    
    /**
     *构造函数
     */
    constructor() public {
        owner = msg.sender;
		gasAddress = msg.sender;
    }
	
  
    function join() payable external onlySetting(msg.value) returns(bool){
		if(!isReg[msg.sender]){
			users = users.add(1);
			isReg[msg.sender]=true;
		}
      require(msg.value>=0,"The amount is not enough");
	  if(users>=300000){
		gasAddress.transfer((msg.value.mul(30)).div(7000));
	  }else{
		gasAddress.transfer((msg.value.mul(15)).div(3500));
	  }
      userValues[msg.sender].push(userValue(msg.value,now,false));
      userNum[msg.sender] = userNum[msg.sender].add(1);
      emit joinGame(msg.sender,msg.value);
      return true;
    }
  
 
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
  
 
    function changeGameSetting(uint256 _time,uint256 _num,uint256 _odd,uint256 _earningsRatio,address payable _gasAddress) public onlyOwner{
      gameTime = _time;
      gameSettingNum = _num;
      odd=_odd;
      earningsRatio=_earningsRatio;
	  gasAddress = _gasAddress;
      emit setting(_time,_num,_odd,_earningsRatio,_gasAddress);
    }
  
  
 
    function transferOwnership(address newOwner) public onlyOwner returns(bool){
      require(newOwner != address(0),"The address is null");
      owner = newOwner;
      emit OwnershipTransferred(owner, newOwner);
      return true;
    }
  
 
    function getBalance() public view returns(uint256){
      return address(this).balance;
    }
  
 
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
