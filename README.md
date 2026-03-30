# BaseRockPaperScissors
BaseRockPaperScissors.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract BaseRockPaperScissors {
    address public immutable owner;
    uint256 public treasuryBalance;

    event GamePlayed(
        address indexed player,
        uint8 playerChoice,     // 0=Rock, 1=Scissors, 2=Paper
        uint8 contractChoice,
        uint256 betAmount,
        uint256 payout,
        uint256 timestamp
    );

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only contract owner can call this");
        _;
    }

    // 0=Rock 1=Scissors 2=Paper
    function play(uint8 _playerChoice) external payable {
        require(_playerChoice <= 2, "Choose 0(Rock), 1(Scissors) or 2(Paper)");
        uint256 bet = msg.value;
        require(bet >= 0.0001 ether && bet <= 0.05 ether, "Bet must be between 0.0001 ~ 0.05 ETH");

        // 伪随机生成合约出拳
        uint256 random = uint256(keccak256(abi.encodePacked(
            block.prevrandao,
            msg.sender,
            block.timestamp,
            block.number
        ))) % 3;

        uint8 contractChoice = uint8(random);

        uint256 payout = 0;
        if (_playerChoice == contractChoice) {
            // 平局：退还本金
            payout = bet;
            payable(msg.sender).transfer(payout);
        } else if (
            (_playerChoice == 0 && contractChoice == 1) || // Rock beats Scissors
            (_playerChoice == 1 && contractChoice == 2) || // Scissors beats Paper
            (_playerChoice == 2 && contractChoice == 0)    // Paper beats Rock
        ) {
            // 赢了：1.9 倍
            payout = bet * 19 / 10;
            payable(msg.sender).transfer(payout);
        } else {
            // 输了：进入金库
            treasuryBalance += bet;
        }

        emit GamePlayed(msg.sender, _playerChoice, contractChoice, bet, payout, block.timestamp);
    }

    function getTreasury() external view returns (uint256) {
        return treasuryBalance;
    }

    function withdrawTreasury() external onlyOwner {
        uint256 amount = treasuryBalance;
        treasuryBalance = 0;
        payable(owner).transfer(amount);
    }

    function getContractInfo() external pure returns (string memory) {
        return "BaseRockPaperScissors - Play Rock Paper Scissors against the contract! 0.0001~0.05 ETH, 1.9x win";
    }
}
