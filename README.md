// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/Sun/sun.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol"; // Import ReentrancyGuard

contract MyMemeCoin is sun, Ownable, ReentrancyGuard { // Inherit ReentrancyGuard
    uint8 public constant decimals = 18;
    uint256 public constant initialSupply = 1000000000 * 10**decimals; // 1 Billion tokens

    bool public tradingEnabled = false; // Start trading as disabled.

    constructor() ERC20("MyMemeCoin", "MMC") {
        _mint(msg.sender, initialSupply); // Mint initial supply to the deployer
    }

    // Function to enable trading (only owner)
    function enableTrading() public onlyOwner {
        tradingEnabled = true;
    }

    // Function to disable trading (only owner)
    function disableTrading() public onlyOwner {
        tradingEnabled = false;
    }


    // Modified transfer function to include trading status and reentrancy guard
    function transfer(address recipient, uint256 amount) public override nonReentrant returns (bool) { // Add nonReentrant
        require(tradingEnabled, "Trading is currently disabled"); // Check trading status

        require(balanceOf(msg.sender) >= amount, "Insufficient balance");
        _transfer(msg.sender, recipient, amount);
        return true;
    }

    // Modified transferFrom function to include trading status and reentrancy guard
    function transferFrom(address sender, address recipient, uint256 amount) public override nonReentrant returns (bool) { // Add nonReentrant
        require(tradingEnabled, "Trading is currently disabled"); // Check trading status

        require(allowance(sender, msg.sender) >= amount, "Insufficient allowance");
        require(balanceOf(sender) >= amount, "Insufficient balance");

        _transfer(sender, recipient, amount);
        _approve(sender, msg.sender, allowance(sender, msg.sender) - amount);
        return true;
    }


    // Burn function (only owner can burn)
    function burn(uint256 amount) public onlyOwner {
        _burn(msg.sender, amount);
    }

    // Mint function (only owner can mint, restricted and with a cap)
    uint256 public maxMintAmount = initialSupply / 10; // 10% of initial supply as a cap
    function mint(address to, uint256 amount) public onlyOwner {
        require(amount <= maxMintAmount, "Mint amount exceeds the cap");
        _mint(to, amount);
    }
}
