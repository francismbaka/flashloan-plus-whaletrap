#// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IERC20 {
    function balanceOf(address account) external view returns (uint256);
}

/**
 * @title CombinedTrap
 * @notice Detects whale transfers and flash-loan-funded transactions.
 */
contract CombinedTrap {
    // The ERC20 token to monitor
    IERC20 public immutable token;

    // Thresholds
    uint256 public immutable whaleThreshold;
    uint256 public immutable flashLoanThreshold;

    // Store pre-balance snapshots for flash loan detection
    mapping(address => uint256) private _preBalances;

    // ───────────────────────── Events ─────────────────────────

    /// Emitted when whale transfer detected
    event WhaleDetected(
        address indexed from,
        address indexed to,
        uint256 amount,
        uint256 blockNumber
    );

    /// Emitted when flash loan detected
    event FlashLoanDetected(
        address indexed user,
        uint256 loanAmount,
        uint256 blockNumber
    );

    // ───────────────────────── Constructor ─────────────────────────

    constructor(
        address _token,
        uint256 _whaleThreshold,
        uint256 _flashLoanThreshold
    ) {
        token = IERC20(_token);
        whaleThreshold = _whaleThreshold;
        flashLoanThreshold = _flashLoanThreshold;
    }

    // ───────────────────────── Whale Trap ─────────────────────────
    /**
     * @notice Call this whenever a transfer occurs to log whales.
     * @dev You’d typically have an off-chain bot or token contract call this.
     */
    function detectWhaleTransfer(
        address from,
        address to,
        uint256 amount
    ) external {
        if (amount >= whaleThreshold) {
            emit WhaleDetected(from, to, amount, block.number);
        }
    }

    // ───────────────────────── Flash Loan Trap ─────────────────────────
    /**
     * @notice Snapshot contract balance before the operation.
     * Call this at the start of your sensitive function.
     */
    function snapshotPreBalance(address user) external {
        _preBalances[user] = token.balanceOf(address(this));
    }

    /**
     * @notice Detect flash loan by comparing balances after the operation.
     * Call this at the end of your sensitive function.
     */
    function detectFlashLoan(address user) external {
        uint256 beforeBal = _preBalances[user];
        uint256 afterBal = token.balanceOf(address(this));
        uint256 incoming = afterBal > beforeBal ? afterBal - beforeBal : 0;

        if (incoming >= flashLoanThreshold) {
            emit FlashLoanDetected(user, incoming, block.number);
        }

        delete _preBalances[user];
    }
}
 flashloan-plus-whaletrap
my whale plus flashloan trap
