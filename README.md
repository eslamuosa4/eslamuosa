# ¿Qué es Base?

Base está construida sobre el OP Stack y hereda la seguridad de Ethereum.

Características principales:
- Gas extremadamente bajo
- Confirmaciones rápidas
- Sin token nativo propio (usa ETH)
- Fuerte enfoque en adopción real de usuarios

Es una de las L2 con mayor crecimiento de actividad.

# Primeros pasos

1. Añade Base a tu wallet (Chain ID: 8453)
2. Consigue ETH en Base
3. Usa Foundry o Hardhat
4. Explora BaseScan: https://basescan.org

Documentación oficial: https://docs.base.org

# Base Guild Builders

Repositorio dedicado a los builders del Guild de Base.

Base utiliza el OP Stack, por lo que es totalmente compatible con Ethereum. Cualquier contrato de Solidity puede desplegarse aquí con costos mínimos.

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleStorage {
    uint256 private value;
    address public lastEditor;

    event ValueChanged(uint256 newValue, address indexed editor);

    function set(uint256 newValue) external {
        value = newValue;
        lastEditor = msg.sender;
        emit ValueChanged(newValue, msg.sender);
    }

    function get() external view returns (uint256) {
        return value;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NameRegistry {
    mapping(address => string) public names;

    event NameSet(address indexed user, string name);

    function setName(string calldata name) external {
        names[msg.sender] = name;
        emit NameSet(msg.sender, name);
    }

    function getName(address user) external view returns (string memory) {
        return names[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Whitelist {
    address public owner;
    mapping(address => bool) public isWhitelisted;

    event Added(address indexed account);
    event Removed(address indexed account);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function add(address account) external onlyOwner {
        isWhitelisted[account] = true;
        emit Added(account);
    }

    function remove(address account) external onlyOwner {
        isWhitelisted[account] = false;
        emit Removed(account);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Faucet {
    address public owner;
    uint256 public amountAllowed = 0.01 ether;
    mapping(address => uint256) public lastRequest;

    event Requested(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function request() external {
        require(block.timestamp >= lastRequest[msg.sender] + 1 days, "Wait 24h");
        require(address(this).balance >= amountAllowed, "Faucet empty");

        lastRequest[msg.sender] = block.timestamp;
        (bool success, ) = msg.sender.call{value: amountAllowed}("");
        require(success, "Transfer failed");
        emit Requested(msg.sender, amountAllowed);
    }

    function donate() external payable {}

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        (bool success, ) = owner.call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }

    receive() external payable {}
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BalanceTracker {
    mapping(address => uint256) public deposits;
    uint256 public totalDeposits;

    event Deposited(address indexed user, uint256 amount);

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        deposits[msg.sender] += msg.value;
        totalDeposits += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function getMyDeposit() external view returns (uint256) {
        return deposits[msg.sender];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Pausable {
    address public owner;
    bool public paused;

    event Paused(address indexed by);
    event Unpaused(address indexed by);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier whenNotPaused() {
        require(!paused, "Contract is paused");
        _;
    }

    function pause() external onlyOwner {
        paused = true;
        emit Paused(msg.sender);
    }

    function unpause() external onlyOwner {
        paused = false;
        emit Unpaused(msg.sender);
    }

    function doSomething() external whenNotPaused {
        // función de ejemplo
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Donation {
    address public beneficiary;
    uint256 public totalDonated;
    mapping(address => uint256) public donations;

    event Donated(address indexed donor, uint256 amount);

    constructor(address _beneficiary) {
        beneficiary = _beneficiary;
    }

    function donate() external payable {
        require(msg.value > 0, "Must send ETH");
        donations[msg.sender] += msg.value;
        totalDonated += msg.value;
        emit Donated(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == beneficiary, "Not beneficiary");
        uint256 amount = address(this).balance;
        (bool success, ) = beneficiary.call{value: amount}("");
        require(success, "Withdraw failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CheckIn {
    mapping(address => uint256) public lastCheckIn;
    mapping(address => uint256) public checkInCount;

    event CheckedIn(address indexed user, uint256 timestamp, uint256 totalCheckIns);

    function checkIn() external {
        lastCheckIn[msg.sender] = block.timestamp;
        checkInCount[msg.sender] += 1;
        emit CheckedIn(msg.sender, block.timestamp, checkInCount[msg.sender]);
    }

    function getCheckInInfo(address user) external view returns (uint256 last, uint256 total) {
        return (lastCheckIn[user], checkInCount[user]);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Deadline {
    uint256 public deadline;
    address public owner;

    event DeadlineSet(uint256 newDeadline);

    constructor(uint256 durationInSeconds) {
        owner = msg.sender;
        deadline = block.timestamp + durationInSeconds;
    }

    function setDeadline(uint256 durationInSeconds) external {
        require(msg.sender == owner, "Not owner");
        deadline = block.timestamp + durationInSeconds;
        emit DeadlineSet(deadline);
    }

    function isExpired() external view returns (bool) {
        return block.timestamp >= deadline;
    }

    function timeLeft() external view returns (uint256) {
        if (block.timestamp >= deadline) return 0;
        return deadline - block.timestamp;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleLottery {
    address[] public players;
    address public winner;
    bool public finished;

    event PlayerJoined(address indexed player);
    event WinnerSelected(address indexed winner);

    function join() external payable {
        require(!finished, "Lottery finished");
        require(msg.value == 0.001 ether, "Send exactly 0.001 ETH");
        players.push(msg.sender);
        emit PlayerJoined(msg.sender);
    }

    function drawWinner() external {
        require(!finished, "Already finished");
        require(players.length > 0, "No players");

        uint256 index = uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, players.length))) % players.length;
        winner = players[index];
        finished = true;

        (bool success, ) = winner.call{value: address(this).balance}("");
        require(success, "Transfer failed");
        emit WinnerSelected(winner);
    }

    function getPlayersCount() external view returns (uint256) {
        return players.length;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Subscription {
    mapping(address => uint256) public expiration;
    uint256 public price = 0.01 ether;
    address public owner;

    event Subscribed(address indexed user, uint256 newExpiration);

    constructor() {
        owner = msg.sender;
    }

    function subscribe() external payable {
        require(msg.value == price, "Incorrect payment");
        uint256 current = expiration[msg.sender];
        uint256 start = current > block.timestamp ? current : block.timestamp;
        expiration[msg.sender] = start + 30 days;
        emit Subscribed(msg.sender, expiration[msg.sender]);
    }

    function isActive(address user) external view returns (bool) {
        return expiration[user] >= block.timestamp;
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        (bool success, ) = owner.call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TimeLock {
    mapping(address => uint256) public unlockTime;
    mapping(address => uint256) public lockedAmount;

    event Locked(address indexed user, uint256 amount, uint256 unlockTime);
    event Withdrawn(address indexed user, uint256 amount);

    function lock(uint256 durationInSeconds) external payable {
        require(msg.value > 0, "Must send ETH");
        require(lockedAmount[msg.sender] == 0, "Already locked");

        lockedAmount[msg.sender] = msg.value;
        unlockTime[msg.sender] = block.timestamp + durationInSeconds;
        emit Locked(msg.sender, msg.value, unlockTime[msg.sender]);
    }

    function withdraw() external {
        require(block.timestamp >= unlockTime[msg.sender], "Still locked");
        require(lockedAmount[msg.sender] > 0, "Nothing to withdraw");

        uint256 amount = lockedAmount[msg.sender];
        lockedAmount[msg.sender] = 0;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Allowance {
    address public owner;
    mapping(address => uint256) public allowance;

    event AllowanceSet(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function setAllowance(address user, uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        allowance[user] = amount;
        emit AllowanceSet(user, amount);
    }

    function withdraw(uint256 amount) external {
        require(allowance[msg.sender] >= amount, "Insufficient allowance");
        require(address(this).balance >= amount, "Insufficient balance");
        allowance[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }

    receive() external payable {}
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PaymentSplitterLite {
    address public payee1;
    address public payee2;

    event PaymentReceived(address indexed from, uint256 amount);
    event PaymentReleased(address indexed to, uint256 amount);

    constructor(address _payee1, address _payee2) {
        require(_payee1 != address(0) && _payee2 != address(0), "Invalid address");
        payee1 = _payee1;
        payee2 = _payee2;
    }

    receive() external payable {
        emit PaymentReceived(msg.sender, msg.value);
    }

    function release() external {
        uint256 balance = address(this).balance;
        require(balance > 0, "Nothing to release");

        uint256 half = balance / 2;
        uint256 rest = balance - half;

        (bool success1, ) = payee1.call{value: half}("");
        require(success1, "Transfer to payee1 failed");
        emit PaymentReleased(payee1, half);

        (bool success2, ) = payee2.call{value: rest}("");
        require(success2, "Transfer to payee2 failed");
        emit PaymentReleased(payee2, rest);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleAuction {
    address public highestBidder;
    uint256 public highestBid;
    address public owner;
    bool public ended;

    event BidPlaced(address indexed bidder, uint256 amount);
    event AuctionEnded(address winner, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function bid() external payable {
        require(!ended, "Auction ended");
        require(msg.value > highestBid, "Bid too low");

        if (highestBidder != address(0)) {
            (bool success, ) = highestBidder.call{value: highestBid}("");
            require(success, "Refund failed");
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
        emit BidPlaced(msg.sender, msg.value);
    }

    function endAuction() external {
        require(msg.sender == owner, "Not owner");
        require(!ended, "Already ended");
        ended = true;

        if (highestBidder != address(0)) {
            (bool success, ) = owner.call{value: highestBid}("");
            require(success, "Transfer failed");
        }
        emit AuctionEnded(highestBidder, highestBid);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract DepositTracker {
    mapping(address => uint256) public totalDeposited;
    mapping(address => uint256) public depositCount;

    event Deposited(address indexed user, uint256 amount, uint256 total);

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        totalDeposited[msg.sender] += msg.value;
        depositCount[msg.sender] += 1;
        emit Deposited(msg.sender, msg.value, totalDeposited[msg.sender]);
    }

    function getInfo(address user) external view returns (uint256 total, uint256 count) {
        return (totalDeposited[user], depositCount[user]);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SharedWallet {
    address public owner;
    mapping(address => bool) public members;

    event MemberAdded(address indexed member);
    event MemberRemoved(address indexed member);
    event Withdrawn(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
        members[msg.sender] = true;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier onlyMember() {
        require(members[msg.sender], "Not member");
        _;
    }

    function addMember(address member) external onlyOwner {
        members[member] = true;
        emit MemberAdded(member);
    }

    function removeMember(address member) external onlyOwner {
        members[member] = false;
        emit MemberRemoved(member);
    }

    function withdraw(uint256 amount) external onlyMember {
        require(address(this).balance >= amount, "Insufficient balance");
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }

    receive() external payable {}
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TipJarMulti {
    address public owner;
    mapping(address => uint256) public tipsReceived;

    event Tipped(address indexed from, address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function tip(address to) external payable {
        require(msg.value > 0, "Must send ETH");
        require(to != address(0), "Invalid address");
        tipsReceived[to] += msg.value;
        emit Tipped(msg.sender, to, msg.value);
    }

    function withdrawTips() external {
        uint256 amount = tipsReceived[msg.sender];
        require(amount > 0, "No tips");
        tipsReceived[msg.sender] = 0;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract EmergencyStop {
    address public owner;
    bool public stopped;

    event Stopped(address indexed by);
    event Resumed(address indexed by);

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    modifier notStopped() {
        require(!stopped, "Contract is stopped");
        _;
    }

    function stop() external onlyOwner {
        stopped = true;
        emit Stopped(msg.sender);
    }

    function resume() external onlyOwner {
        stopped = false;
        emit Resumed(msg.sender);
    }

    function criticalAction() external notStopped {
        // acción de ejemplo
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleBridge {
    address public owner;
    mapping(address => uint256) public locked;

    event Locked(address indexed user, uint256 amount);
    event Unlocked(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function lock() external payable {
        require(msg.value > 0, "Must send ETH");
        locked[msg.sender] += msg.value;
        emit Locked(msg.sender, msg.value);
    }

    function unlock(address user, uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        require(locked[user] >= amount, "Insufficient locked");
        locked[user] -= amount;
        (bool success, ) = user.call{value: amount}("");
        require(success, "Transfer failed");
        emit Unlocked(user, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Refundable {
    mapping(address => uint256) public deposits;
    address public owner;

    event Deposited(address indexed user, uint256 amount);
    event Refunded(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        deposits[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function refund() external {
        uint256 amount = deposits[msg.sender];
        require(amount > 0, "Nothing to refund");
        deposits[msg.sender] = 0;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Refund failed");
        emit Refunded(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FixedDeposit {
    struct Deposit {
        uint256 amount;
        uint256 unlockTime;
    }

    mapping(address => Deposit) public deposits;

    event Deposited(address indexed user, uint256 amount, uint256 unlockTime);
    event Withdrawn(address indexed user, uint256 amount);

    function deposit(uint256 lockDays) external payable {
        require(msg.value > 0, "Must send ETH");
        require(deposits[msg.sender].amount == 0, "Already has deposit");
        require(lockDays > 0 && lockDays <= 365, "Invalid days");

        uint256 unlock = block.timestamp + (lockDays * 1 days);
        deposits[msg.sender] = Deposit(msg.value, unlock);
        emit Deposited(msg.sender, msg.value, unlock);
    }

    function withdraw() external {
        Deposit memory d = deposits[msg.sender];
        require(d.amount > 0, "No deposit");
        require(block.timestamp >= d.unlockTime, "Still locked");

        delete deposits[msg.sender];
        (bool success, ) = msg.sender.call{value: d.amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, d.amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GasTank {
    mapping(address => uint256) public balance;
    address public owner;

    event Filled(address indexed user, uint256 amount);
    event Used(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function fill() external payable {
        require(msg.value > 0, "Must send ETH");
        balance[msg.sender] += msg.value;
        emit Filled(msg.sender, msg.value);
    }

    function use(uint256 amount) external {
        require(balance[msg.sender] >= amount, "Insufficient balance");
        balance[msg.sender] -= amount;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Used(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Bounty {
    address public creator;
    address public solver;
    uint256 public reward;
    bool public claimed;
    string public description;

    event BountyCreated(address indexed creator, uint256 reward, string description);
    event BountyClaimed(address indexed solver, uint256 reward);

    constructor(string memory _description) payable {
        require(msg.value > 0, "Reward required");
        creator = msg.sender;
        reward = msg.value;
        description = _description;
        emit BountyCreated(msg.sender, msg.value, _description);
    }

    function claim() external {
        require(!claimed, "Already claimed");
        require(msg.sender != creator, "Creator cannot claim");
        claimed = true;
        solver = msg.sender;
        (bool success, ) = msg.sender.call{value: reward}("");
        require(success, "Transfer failed");
        emit BountyClaimed(msg.sender, reward);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Lockbox {
    address public owner;
    uint256 public unlockTime;
    bool public locked = true;

    event Locked(uint256 unlockTime);
    event Unlocked(address indexed by);

    constructor(uint256 lockDuration) {
        owner = msg.sender;
        unlockTime = block.timestamp + lockDuration;
        emit Locked(unlockTime);
    }

    function unlock() external {
        require(msg.sender == owner, "Not owner");
        require(block.timestamp >= unlockTime, "Still locked");
        require(locked, "Already unlocked");
        locked = false;
        emit Unlocked(msg.sender);
    }

    function isLocked() external view returns (bool) {
        return locked && block.timestamp < unlockTime;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PiggyBank {
    address public owner;
    uint256 public goal;
    uint256 public totalSaved;

    event Deposited(address indexed from, uint256 amount);
    event GoalReached(uint256 total);
    event Withdrawn(uint256 amount);

    constructor(uint256 _goal) {
        owner = msg.sender;
        goal = _goal;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        totalSaved += msg.value;
        emit Deposited(msg.sender, msg.value);

        if (totalSaved >= goal) {
            emit GoalReached(totalSaved);
        }
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        require(totalSaved >= goal, "Goal not reached");
        uint256 amount = address(this).balance;
        totalSaved = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SharedCounter {
    uint256 public count;
    mapping(address => uint256) public contributions;

    event Incremented(address indexed user, uint256 newCount);

    function increment() external {
        count += 1;
        contributions[msg.sender] += 1;
        emit Incremented(msg.sender, count);
    }

    function getContribution(address user) external view returns (uint256) {
        return contributions[user];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Pledge {
    mapping(address => uint256) public pledges;
    uint256 public totalPledged;
    address public beneficiary;
    bool public claimed;

    event Pledged(address indexed user, uint256 amount);
    event Claimed(uint256 amount);

    constructor(address _beneficiary) {
        beneficiary = _beneficiary;
    }

    function pledge() external payable {
        require(msg.value > 0, "Must send ETH");
        require(!claimed, "Already claimed");
        pledges[msg.sender] += msg.value;
        totalPledged += msg.value;
        emit Pledged(msg.sender, msg.value);
    }

    function claim() external {
        require(msg.sender == beneficiary, "Not beneficiary");
        require(!claimed, "Already claimed");
        claimed = true;
        uint256 amount = address(this).balance;
        (bool success, ) = beneficiary.call{value: amount}("");
        require(success, "Transfer failed");
        emit Claimed(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract EthLogger {
    address public owner;

    event EthReceived(address indexed from, uint256 amount, uint256 timestamp);
    event EthWithdrawn(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    receive() external payable {
        emit EthReceived(msg.sender, msg.value, block.timestamp);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Withdraw failed");
        emit EthWithdrawn(owner, amount);
    }

    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleNFTCounter {
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(uint256 => address) public ownerOf;

    event Minted(address indexed to, uint256 tokenId);

    function mint() external {
        uint256 tokenId = totalSupply;
        totalSupply += 1;
        balanceOf[msg.sender] += 1;
        ownerOf[tokenId] = msg.sender;
        emit Minted(msg.sender, tokenId);
    }

    function getOwner(uint256 tokenId) external view returns (address) {
        return ownerOf[tokenId];
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Collectible {
    mapping(address => mapping(uint256 => bool)) public owns;
    mapping(uint256 => address) public creator;
    uint256 public nextId = 1;

    event Created(uint256 indexed id, address indexed creator);
    event Claimed(uint256 indexed id, address indexed claimer);

    function create() external returns (uint256) {
        uint256 id = nextId;
        nextId += 1;
        creator[id] = msg.sender;
        owns[msg.sender][id] = true;
        emit Created(id, msg.sender);
        return id;
    }

    function claim(uint256 id) external {
        require(creator[id] != address(0), "Does not exist");
        require(!owns[msg.sender][id], "Already owns");
        owns[msg.sender][id] = true;
        emit Claimed(id, msg.sender);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract DonationPool {
    address public owner;
    uint256 public totalDonations;
    mapping(address => uint256) public donations;

    event Donated(address indexed donor, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function donate() external payable {
        require(msg.value > 0, "Must send ETH");
        donations[msg.sender] += msg.value;
        totalDonations += msg.value;
        emit Donated(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Withdraw failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RewardPool {
    address public owner;
    mapping(address => uint256) public rewards;

    event RewardAssigned(address indexed user, uint256 amount);
    event RewardClaimed(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function assignReward(address user, uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        rewards[user] += amount;
        emit RewardAssigned(user, amount);
    }

    function claimReward() external {
        uint256 amount = rewards[msg.sender];
        require(amount > 0, "No rewards");
        require(address(this).balance >= amount, "Insufficient pool");
        rewards[msg.sender] = 0;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit RewardClaimed(msg.sender, amount);
    }

    receive() external payable {}
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TreasuryLite {
    address public owner;
    uint256 public totalReceived;

    event Received(address indexed from, uint256 amount);
    event Spent(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    receive() external payable {
        totalReceived += msg.value;
        emit Received(msg.sender, msg.value);
    }

    function spend(address to, uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        require(address(this).balance >= amount, "Insufficient funds");
        (bool success, ) = to.call{value: amount}("");
        require(success, "Transfer failed");
        emit Spent(to, amount);
    }

    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SplitPayment {
    address public recipient1;
    address public recipient2;
    uint256 public share1;

    event PaymentSplit(address indexed from, uint256 amount);

    constructor(address _r1, address _r2, uint256 _share1) {
        require(_share1 <= 100, "Invalid share");
        recipient1 = _r1;
        recipient2 = _r2;
        share1 = _share1;
    }

    receive() external payable {
        uint256 amount1 = (msg.value * share1) / 100;
        uint256 amount2 = msg.value - amount1;

        (bool s1, ) = recipient1.call{value: amount1}("");
        require(s1, "Transfer 1 failed");
        (bool s2, ) = recipient2.call{value: amount2}("");
        require(s2, "Transfer 2 failed");

        emit PaymentSplit(msg.sender, msg.value);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleVaultETH {
    mapping(address => uint256) public deposits;
    uint256 public totalDeposits;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        deposits[msg.sender] += msg.value;
        totalDeposits += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(deposits[msg.sender] >= amount, "Insufficient balance");
        deposits[msg.sender] -= amount;
        totalDeposits -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FeeWallet {
    address public owner;
    uint256 public feePercent = 5;
    uint256 public collectedFees;

    event PaymentReceived(address indexed from, uint256 amount, uint256 fee);
    event FeesWithdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    receive() external payable {
        uint256 fee = (msg.value * feePercent) / 100;
        collectedFees += fee;
        emit PaymentReceived(msg.sender, msg.value, fee);
    }

    function withdrawFees() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = collectedFees;
        collectedFees = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit FeesWithdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeDeposit {
    mapping(address => uint256) public balances;
    address public guardian;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    constructor(address _guardian) {
        guardian = _guardian;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }

    function emergencyWithdraw(address user) external {
        require(msg.sender == guardian, "Not guardian");
        uint256 amount = balances[user];
        balances[user] = 0;
        (bool success, ) = user.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MiniTreasury {
    address public owner;
    uint256 public balance;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        balance += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        require(balance >= amount, "Insufficient balance");
        balance -= amount;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(owner, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ETHPool {
    address public owner;
    uint256 public totalPool;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed to, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        totalPool += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(msg.sender == owner, "Not owner");
        require(totalPool >= amount, "Insufficient pool");
        totalPool -= amount;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(owner, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TipPool {
    address public owner;
    uint256 public totalTips;

    event Tipped(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function tip() external payable {
        require(msg.value > 0, "Must send ETH");
        totalTips += msg.value;
        emit Tipped(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MiniBank {
    mapping(address => uint256) public balances;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CoinJar {
    address public owner;
    uint256 public total;

    event Added(address indexed from, uint256 amount);
    event Emptied(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function add() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Added(msg.sender, msg.value);
    }

    function empty() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Emptied(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Box {
    address public owner;
    uint256 public total;

    event Filled(address indexed from, uint256 amount);
    event Emptied(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function fill() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Filled(msg.sender, msg.value);
    }

    function empty() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Emptied(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VaultLite {
    address public owner;
    mapping(address => uint256) public balances;

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(msg.sender, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Bucket {
    address public owner;
    uint256 public filled;

    event Filled(address indexed from, uint256 amount);
    event Emptied(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function fill() external payable {
        require(msg.value > 0, "Must send ETH");
        filled += msg.value;
        emit Filled(msg.sender, msg.value);
    }

    function empty() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        filled = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Emptied(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Safe {
    address public owner;
    uint256 public balance;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        balance += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        balance = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VaultBox {
    address public owner;
    uint256 public stored;

    event Stored(address indexed from, uint256 amount);
    event Released(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function store() external payable {
        require(msg.value > 0, "Must send ETH");
        stored += msg.value;
        emit Stored(msg.sender, msg.value);
    }

    function release() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        stored = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Released(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BankBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Hold {
    address public owner;
    uint256 public held;

    event Held(address indexed from, uint256 amount);
    event Released(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function hold() external payable {
        require(msg.value > 0, "Must send ETH");
        held += msg.value;
        emit Held(msg.sender, msg.value);
    }

    function release() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        held = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Released(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Store {
    address public owner;
    uint256 public stored;

    event Stored(address indexed from, uint256 amount);
    event Released(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function store() external payable {
        require(msg.value > 0, "Must send ETH");
        stored += msg.value;
        emit Stored(msg.sender, msg.value);
    }

    function release() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        stored = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Released(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Cache {
    address public owner;
    uint256 public cached;

    event Cached(address indexed from, uint256 amount);
    event Cleared(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function cache() external payable {
        require(msg.value > 0, "Must send ETH");
        cached += msg.value;
        emit Cached(msg.sender, msg.value);
    }

    function clear() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        cached = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Cleared(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PocketVault {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Purse {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ChestBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Locker {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VaultSafe {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PocketSafe {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ReserveBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VaultHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FundBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CofferBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LockSafe {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeHoldBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TreasureBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ReserveSafe {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ChestSafe {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PocketHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VaultHoldBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeHoldLite {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BoxHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LockerHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TreasureHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ReserveHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(succ// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ChestHold {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PocketBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VaultLiteBox {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeMini {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    // SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BoxMini {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LockerMini {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TreasureMini {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ChestMini {
    address public owner;
    uint256 public total;

    event Deposited(address indexed from, uint256 amount);
    event Withdrawn(uint256 amount);

    constructor() {
        owner = msg.sender;
    }

    function deposit() external payable {
        require(msg.value > 0, "Must send ETH");
        total += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw() external {
        require(msg.sender == owner, "Not owner");
        uint256 amount = address(this).balance;
        total = 0;
        (bool success, ) = owner.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(amount);
    }
}
