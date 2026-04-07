# DailyMessageBoard
DailyMessageBoard
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

/// @title Daily Message Board on Base Chain
/// @notice A simple and original contract where users can post messages and boost them with ETH
contract DailyMessageBoard {

    // Message structure
    struct Message {
        string content;      // The message text
        address author;      // Who posted the message
        uint256 score;       // Score increased by ETH payments
        uint256 timestamp;   // Time when the message was posted or last updated
    }

    // Array to store all messages
    Message[] public allMessages;

    // Mapping to track the last message index of each user
    mapping(address => uint256) public lastMessageIndex;

    // Events
    event MessagePosted(uint256 indexed messageId, address author, string content, uint256 initialScore);
    event ScoreIncreased(uint256 indexed messageId, uint256 addedAmount, uint256 newScore);

    // Constructor
    constructor() {
        // No special initialization needed
    }

    /// @notice Post a new message or update your existing one. Minimum 0.0001 ETH required.
    /// @param _content The content of the message (max 280 characters)
    function postMessage(string calldata _content) external payable {
        require(bytes(_content).length > 0 && bytes(_content).length <= 280, "Message must be between 1 and 280 characters");
        require(msg.value >= 0.0001 ether, "At least 0.0001 ETH is required as initial score");

        uint256 currentIndex;

        if (lastMessageIndex[msg.sender] == 0) {
            // New message
            currentIndex = allMessages.length;
            allMessages.push(Message({
                content: _content,
                author: msg.sender,
                score: msg.value,
                timestamp: block.timestamp
            }));
        } else {
            // Update existing message
            currentIndex = lastMessageIndex[msg.sender] - 1;
            allMessages[currentIndex].content = _content;
            allMessages[currentIndex].score += msg.value;
            allMessages[currentIndex].timestamp = block.timestamp;
        }

        lastMessageIndex[msg.sender] = currentIndex + 1;

        emit MessagePosted(currentIndex, msg.sender, _content, msg.value);
    }

    /// @notice Increase the score of a specific message by sending ETH
    /// @param _messageId The index of the message to boost
    function increaseScore(uint256 _messageId) external payable {
        require(_messageId < allMessages.length, "Message does not exist");
        require(msg.value > 0, "Must send ETH to increase score");

        allMessages[_messageId].score += msg.value;
        allMessages[_messageId].timestamp = block.timestamp;

        emit ScoreIncreased(_messageId, msg.value, allMessages[_messageId].score);
    }

    /// @notice Get the message with the highest score
    /// @return The highest scoring message
    function getTopMessage() external view returns (Message memory) {
        require(allMessages.length > 0, "No messages posted yet");

        uint256 topIndex = 0;
        uint256 highestScore = allMessages[0].score;

        for (uint256 i = 1; i < allMessages.length; i++) {
            if (allMessages[i].score > highestScore) {
                highestScore = allMessages[i].score;
                topIndex = i;
            }
        }

        return allMessages[topIndex];
    }

    /// @notice Get the last message posted by a specific user
    /// @param _user Address of the user
    function getLastMessageOf(address _user) external view returns (Message memory) {
        uint256 index = lastMessageIndex[_user];
        require(index > 0, "User has not posted any message yet");
        return allMessages[index - 1];
    }

    /// @notice Get the total number of messages
    function getTotalMessages() external view returns (uint256) {
        return allMessages.length;
    }

    /// @notice Withdraw all ETH accumulated in the contract (only direct caller)
    function withdrawFunds() external {
        require(msg.sender == tx.origin, "Only direct caller can withdraw");

        uint256 balance = address(this).balance;
        require(balance > 0, "No ETH available in the contract");

        (bool success, ) = payable(msg.sender).call{value: balance}("");
        require(success, "ETH withdrawal failed");
    }
}
#Remix IDE (remix.ethereum.org)

