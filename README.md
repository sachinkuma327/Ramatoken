// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract RAMAToken is ERC20, Ownable {
    // Mapping for email verification status
    mapping(address => bool) public isVerified;

    // Events for verification and transfer approval
    event Verified(address indexed user);
    event TransferAttempt(address indexed from, address indexed to, uint256 amount);

    constructor() ERC20("RAMAToken", "RAMA") {
        // Mint initial supply
        uint256 totalSupply = 10_000_000_000 * 10**decimals();
        uint256 ownerSupply = 2_000_000_000 * 10**decimals();
        _mint(msg.sender, ownerSupply); // 2 billion tokens to owner
        _mint(address(this), totalSupply - ownerSupply); // 8 billion tokens for public
    }

    // Function to verify users
    function verifyUser(address user) external onlyOwner {
        isVerified[user] = true;
        emit Verified(user);
    }

    // Override transfer to include verification check
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override {
        require(
            isVerified[from] || from == address(0), // Allow minting without verification
            "Sender is not verified"
        );
        super._beforeTokenTransfer(from, to, amount);
    }

    // Mint function restricted to admin
    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }

    // Public sale function to distribute tokens
    function distribute(address to, uint256 amount) external onlyOwner {
        require(balanceOf(address(this)) >= amount, "Insufficient public tokens");
        _transfer(address(this), to, amount);
    }
}
