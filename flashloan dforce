// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
import "IFlashLoanRecipient.sol";
import "IBalancerVault.sol";

interface Approval {
    function approve (address spender, uint256 rawAmount) external;
}

interface Dforce {
    function approve (address spender, uint256 rawAmount) external;
    function balanceOf(address) external view returns (uint256 balance);
    function mint(address _recipient, uint256 _mintAmount) external;
    function redeem(address _from, uint256 _redeemiToken) external;
}

contract BalancerFlashLoan is IFlashLoanRecipient {
    using SafeMath for uint256;
    using SafeMath for uint16;
    IERC20 public token;
    
    address private dContractAddress = 0xec85F77104Ffa35a5411750d70eDFf8f1496d95b; // Reemplaza con la dirección real
    address private daiContractAddress = 0x8f3Cf7ad23Cd3CaDbD9735AFf958023239c6A063;
    address private yoContractAddress = address(this);
    address public immutable vault;
    address private owner;

    constructor(address _vault) {
        vault = _vault;
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the contract owner can call this function");
        _;
    }

        function receiveFlashLoan(
        IERC20[] memory tokens,
        uint256[] memory amounts,
        uint256[] memory feeAmounts,
        bytes memory
        ) external override {
        for (uint256 i = 0; i < tokens.length; ++i) {
            IERC20 token = tokens[i];
            uint256 amount = amounts[i];
            uint256 feeAmount = feeAmounts[i];

            executemint();
            executeredeem();

            token.transfer(vault, amount);
        }
    }

    function flashLoan(
        IERC20[] memory tokens,
        uint256[] memory amounts,
        bytes memory userData
    ) external {
        IBalancerVault(vault).flashLoan(
            IFlashLoanRecipient(address(this)),
            tokens,
            amounts,
            userData
        );
    }

    function executemint() public payable {
        Approval dai = Approval(daiContractAddress);
        dai.approve(0xBA12222222228d8Ba445958a75a0704d566BF2C8, 900000000000000000000);
        dai.approve(address(this), 900000000000000000000);
        dai.approve(dContractAddress, 900000000000000000000);
        Dforce d = Dforce(dContractAddress);
        d.mint(address(this), 100000000000000000000);
    }

    function executeredeem() public {
        Dforce d = Dforce(dContractAddress);
        d.approve(address(this), d.balanceOf(address(this)));
        d.balanceOf(address(this));
        d.redeem(address(this), d.balanceOf(address(this)));
    }

    function getDaiBalance() external view returns (uint256) {
    IERC20 dai = IERC20(daiContractAddress);
    return dai.balanceOf(address(this));
    }

        function approve(address spender, IERC20[] memory tokens, uint256[] memory amounts) external {
        require(tokens.length == amounts.length, "Token and amount arrays must have the same length");

        for (uint256 i = 0; i < tokens.length; i++) {
            tokens[i].approve(spender, amounts[i]);
        }
    }

    function withdrawERC20(IERC20[] memory tokens, uint256[] memory amounts) external onlyOwner {
        require(tokens.length == amounts.length, "Token and amount arrays must have the same length");

        for (uint256 i = 0; i < tokens.length; i++) {
            uint256 balance = tokens[i].balanceOf(address(this));
            require(balance >= amounts[i], "Insufficient token balance");

            tokens[i].transfer(msg.sender, amounts[i]);
        }
    }
    
    receive() external payable {
        // Esta función permite que el contrato reciba Ether cuando se le envía directamente.
    }
}
