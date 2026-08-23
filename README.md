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
