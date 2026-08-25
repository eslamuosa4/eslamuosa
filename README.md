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
