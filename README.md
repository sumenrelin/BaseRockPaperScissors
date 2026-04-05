# BaseRockPaperScissors
BaseRockPaperScissors.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

event GamePlayed(
    address indexed player,
    uint8 playerChoice,     // 0=グー, 1=チョキ, 2=パー
    uint8 contractChoice,
    uint256 betAmount,
    uint256 payout,
    uint256 timestamp
);

constructor() {
    owner = msg.sender;
}

modifier onlyOwner() {
    require(msg.sender == owner, unicode"コントラクト所有者のみ呼び出せます");
    _;
}

// 0=グー 1=チョキ 2=パー
function play(uint8 _playerChoice) external payable {
    require(_playerChoice <= 2, unicode"0（グー）、1（チョキ）、2（パー）のいずれかを選択してください");
    uint256 bet = msg.value;
    require(bet >= 0.0001 ether && bet <= 0.05 ether, unicode"ベット金額は 0.0001 ETH 〜 0.05 ETH の間で入力してください");

    // 擬似ランダムでコントラクトの手を生成
    uint256 random = uint256(keccak256(abi.encodePacked(
        block.prevrandao,
        msg.sender,
        block.timestamp,
        block.number
    ))) % 3;

    uint8 contractChoice = uint8(random);

    uint256 payout = 0;
    if (_playerChoice == contractChoice) {
        // あいこ：賭け金をそのまま返金
        payout = bet;
        payable(msg.sender).transfer(payout);
    } else if (
        (_playerChoice == 0 && contractChoice == 1) || // グーはチョキに勝つ
        (_playerChoice == 1 && contractChoice == 2) || // チョキはパーに勝つ
        (_playerChoice == 2 && contractChoice == 0)    // パーはグーに勝つ
    ) {
        // 勝利：1.9倍
        payout = bet * 19 / 10;
        payable(msg.sender).transfer(payout);
    } else {
        // 負け：宝箱（treasury）に入る
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
    return "BaseRockPaperScissors - コントラクトとじゃんけん対戦！ 0.0001〜0.05 ETH、勝利で1.9倍";
}
