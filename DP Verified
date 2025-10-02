// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

/* ================================================
OpenZeppelin-like (flattened)
Context, Ownable, ReentrancyGuard, ERC20
=================================================*/

// ============ Context ============
abstract contract Context {
    function _msgSender() internal view virtual returns (address) {
        return msg.sender;
    }
    function _msgData() internal view virtual returns (bytes calldata) {
        return msg.data;
    }
}

// ============ Ownable ============
abstract contract Ownable is Context {
    address private _owner;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    constructor() {
        _transferOwnership(_msgSender());
    }

    modifier onlyOwner() {
        require(owner() == _msgSender(), "Ownable: caller is not the owner");
        _;
    }

    function owner() public view virtual returns (address) {
        return _owner;
    }

    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is zero");
        _transferOwnership(newOwner);
    }

    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
}

// ============ ReentrancyGuard ============
abstract contract ReentrancyGuard {
    uint256 private _status;

    constructor() {
        _status = 1;
    }

    modifier nonReentrant() {
        require(_status != 2, "ReentrancyGuard: reentrant call");
        _status = 2;
        _;
        _status = 1;
    }
}

// ============ IERC20 ============
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

// ============ ERC20 ============
contract ERC20 is Context, IERC20 {
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    uint256 private _totalSupply;

    string private _name;
    string private _symbol;

    constructor(string memory name_, string memory symbol_) {
        _name = name_;
        _symbol = symbol_;
    }

    function name() public view returns (string memory) { return _name; }
    function symbol() public view returns (string memory) { return _symbol; }
    function decimals() public pure returns (uint8) { return 18; }
    function totalSupply() public view override returns (uint256) { return _totalSupply; }
    function balanceOf(address account) public view override returns (uint256) { return _balances[account]; }

    function transfer(address recipient, uint256 amount) public override returns (bool) {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }

    function allowance(address owner, address spender) public view override returns (uint256) {
        return _allowances[owner][spender];
    }

    function approve(address spender, uint256 amount) public override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
        uint256 currentAllowance = _allowances[sender][_msgSender()];
        require(currentAllowance >= amount, "ERC20: transfer exceeds allowance");
        _transfer(sender, recipient, amount);
        _approve(sender, _msgSender(), currentAllowance - amount);
        return true;
    }

    function _transfer(address sender, address recipient, uint256 amount) internal virtual {
        require(sender != address(0), "ERC20: sender zero");
        require(recipient != address(0), "ERC20: recipient zero");

        uint256 senderBalance = _balances[sender];
        require(senderBalance >= amount, "ERC20: transfer exceeds balance");

        _balances[sender] = senderBalance - amount;
        _balances[recipient] += amount;
        emit Transfer(sender, recipient, amount);
    }

    function _mint(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: mint to zero");
        _totalSupply += amount;
        _balances[account] += amount;
        emit Transfer(address(0), account, amount);
    }

    function _burn(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: burn from zero");
        uint256 accountBalance = _balances[account];
        require(accountBalance >= amount, "ERC20: burn exceeds balance");
        _balances[account] = accountBalance - amount;
        _totalSupply -= amount;
        emit Transfer(account, address(0), amount);
    }

    function _approve(address owner, address spender, uint256 amount) internal virtual {
        require(owner != address(0), "ERC20: approve from zero");
        require(spender != address(0), "ERC20: approve to zero");
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
}

// ================= Diamond Panda Contract =================
contract TestDirect is ERC20, Ownable, ReentrancyGuard {
    uint256 public constant MAX_SUPPLY = 21_000_000 * 10**18;
    uint256 public minContribution = 0.01 ether;
    uint256 public maxContribution = 10000 ether;

    mapping(uint256 => uint256) public blockTotalETH;

    event Contribution(address indexed contributor, uint256 amountETH, uint256 rewardTokens);
    event EmergencyWithdraw(address indexed to, uint256 amount);

    constructor() ERC20("Diamond Panda", "DP") {
        _mint(msg.sender, 840_000 * 10**18); // premint 840,000 DP
    }

    receive() external payable nonReentrant {
        require(msg.value >= minContribution, "Below minimum contribution");
        require(msg.value <= maxContribution, "Exceeds max contribution");

        uint256 reward = getCurrentBlockReward();
        require(totalSupply() + reward <= MAX_SUPPLY, "Exceeds max supply");

        _mint(msg.sender, reward);
        blockTotalETH[block.number] += msg.value;

        emit Contribution(msg.sender, msg.value, reward);
    }

    function getCurrentBlockReward() public view returns (uint256) {
        uint256 supply = totalSupply();
        if (supply < 10_500_000 * 10**18) return 2 * 10**18;
        else if (supply < 15_750_000 * 10**18) return 1 * 10**18;
        else if (supply < 18_375_000 * 10**18) return 5e17;
        else if (supply < 19_687_500 * 10**18) return 25e16;
        else if (supply < 20_343_750 * 10**18) return 125e15;
        else if (supply < 20_671_875 * 10**18) return 62500000000000000;
        else if (supply < 20_835_937 * 10**18) return 31250000000000000;
        else if (supply < 20_917_968 * 10**18) return 15625000000000000;
        else if (supply < 20_958_984 * 10**18) return 7812500000000000;
        else if (supply < 20_979_492 * 10**18) return 3906250000000000;
        else if (supply < 20_989_746 * 10**18) return 1953125000000000;
        else if (supply < 20_994_873 * 10**18) return 976562500000000;
        else if (supply < 20_997_436 * 10**18) return 488281250000000;
        else if (supply < 20_998_718 * 10**18) return 244140625000000;
        else if (supply < 20_999_359 * 10**18) return 122070312500000;
        else if (supply < 20_999_679 * 10**18) return 61035156250000;
        else if (supply < 20_999_839 * 10**18) return 30517578125000;
        else if (supply < 20_999_919 * 10**18) return 15258789062500;
        else if (supply < 20_999_959 * 10**18) return 7629394531250;
        else if (supply < 20_999_979 * 10**18) return 3814697265625;
        else return 1907348632812;
    }

    function emergencyWithdrawETH(address payable to, uint256 amount) external onlyOwner {
        require(address(this).balance >= amount, "Not enough 0G");
        to.transfer(amount);
        emit EmergencyWithdraw(to, amount);
    }

    function setContributionLimits(uint256 minAmount, uint256 maxAmount) external onlyOwner {
        minContribution = minAmount;
        maxContribution = maxAmount;
    }
}
