# voting-contract
This is public voting contract


```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Voting {
    // Struct for each voting option with a description, party name, and vote count
    struct Option {
        string description;
        string party;
        uint voteCount;
    }

    // Struct to define each voting round
    struct VotingRound {
        bool active;
        uint endTime;
        Option[] options; // Array of options in this round
        mapping(address => bool) hasVoted; // Tracks if an address has voted in this round
    }

    // Mapping from round ID to VotingRound details
    mapping(uint => VotingRound) public votingRounds;
    uint public currentRoundId;
    
    // Address of the contract owner (admin)
    address public owner;

    // Constructor to set the contract owner
    constructor() {
        owner = msg.sender;
    }

    /**
     * @notice Create a new voting round with descriptions and party names for each option
     * @param optionDescriptions Array of option descriptions
     * @param partyNames Array of party names corresponding to each option
     * @param _durationInMinutes Duration of the voting round in minutes
     */
    function createVotingRound(
        string[] calldata optionDescriptions,
        string[] calldata partyNames,
        uint _durationInMinutes
    ) external {
        require(msg.sender == owner, "Not the contract owner");
        require(optionDescriptions.length > 1, "Need at least 2 options");
        require(optionDescriptions.length == partyNames.length, "Descriptions and parties length mismatch");

        VotingRound storage newRound = votingRounds[currentRoundId];
        newRound.active = true;
        newRound.endTime = block.timestamp + (_durationInMinutes * 1 minutes);

        // Add options to the round with descriptions and party names
        for (uint i = 0; i < optionDescriptions.length; i++) {
            newRound.options.push(Option({
                description: optionDescriptions[i],
                party: partyNames[i],
                voteCount: 0
            }));
        }

        currentRoundId++; // Move to the next round ID for future rounds
    }

    /**
     * @notice Cast a vote in a specific round
     * @param roundId The ID of the voting round
     * @param optionIndex The index of the chosen option
     */
    function vote(uint roundId, uint optionIndex) external {
        VotingRound storage round = votingRounds[roundId];
        require(round.active, "Voting has ended for this round");
        require(block.timestamp < round.endTime, "Voting time expired");
        require(optionIndex < round.options.length, "Invalid option");
        require(!round.hasVoted[msg.sender], "Already voted in this round");

        // Record the vote
        round.options[optionIndex].voteCount++;
        round.hasVoted[msg.sender] = true;
    }
    
    /**
     * @notice End voting for a specific round
     * @param roundId The ID of the voting round
     */
    function endVoting(uint roundId) external {
        require(msg.sender == owner, "Not the contract owner");
        VotingRound storage round = votingRounds[roundId];
        require(round.active, "Voting already ended for this round");
        round.active = false;
    }
    
    /**
     * @notice Get details of an option in a specific round
     * @param roundId The ID of the voting round
     * @param optionIndex The index of the option
     * @return description The description of the option
     * @return party The party name of the option
     * @return voteCount The number of votes the option has received
     */
    function getOptionDetails(
        uint roundId,
        uint optionIndex
    ) external view returns (string memory description, string memory party, uint voteCount) {
        VotingRound storage round = votingRounds[roundId];
        require(optionIndex < round.options.length, "Invalid option");

        Option storage option = round.options[optionIndex];
        return (option.description, option.party, option.voteCount);
    }

    /**
     * @notice Fetch all vote counts for a specific round for AI sentiment analysis
     * @param roundId The ID of the voting round
     * @return votes Array of vote counts for each option
     */
    function getAllVotes(uint roundId) external view returns (uint[] memory votes) {
        VotingRound storage round = votingRounds[roundId];
        votes = new uint[](round.options.length);
        for (uint i = 0; i < round.options.length; i++) {
            votes[i] = round.options[i].voteCount;
        }
        return votes;
    }
}
```
