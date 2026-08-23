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
