# Voting_with_Blockchain
Developed a secure and transparent online voting system using Ethereum blockchain technology



// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Election {
    address public admin;
    enum ElectionState { NOT_STARTED, ONGOING, ENDED }
    ElectionState public electionState;
    
    struct Candidate {
        string name;
        string proposal;
        uint votes;
    }
    
    struct Voter {
        string name;
        uint votedFor;  // Candidate ID
        bool hasDelegated;
        address delegate;
    }
    
    mapping(uint => Candidate) public candidates;
    mapping(address => Voter) public voters;
    
    uint public candidateCount;
    uint public voterCount;
    
    event ElectionStarted();
    event ElectionEnded();
    event VoteCast(address indexed voter, uint indexed candidateId);
    event VoteDelegated(address indexed from, address indexed to);
    
    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin can call this function");
        _;
    }
    
    modifier onlyVoter() {
        require(voters[msg.sender].votedFor == 0, "Voter has already voted");
        _;
    }
    
    modifier onlyOngoingElection() {
        require(electionState == ElectionState.ONGOING, "Election is not ongoing");
        _;
    }
    
    constructor() {
        admin = msg.sender;
        electionState = ElectionState.NOT_STARTED;
    }
    
    function addCandidate(string memory _name, string memory _proposal) external onlyAdmin {
        require(electionState == ElectionState.NOT_STARTED, "Election has already started");
        candidateCount++;
        candidates[candidateCount] = Candidate(_name, _proposal, 0);
    }
    
    function addVoter(address _voter) external onlyAdmin {
        require(electionState == ElectionState.NOT_STARTED, "Election has already started");
        require(voters[_voter].votedFor == 0, "Voter already registered");
        voterCount++;
        voters[_voter] = Voter("", 0, false, address(0));
    }
    
    function startElection() external onlyAdmin {
        require(electionState == ElectionState.NOT_STARTED, "Election has already started");
        electionState = ElectionState.ONGOING;
        emit ElectionStarted();
    }
    
    function displayCandidateDetails(uint _id) external view returns (uint, string memory, string memory) {
        return (_id, candidates[_id].name, candidates[_id].proposal);
    }
    
    function showWinner() external view returns (string memory, uint, uint) {
        require(electionState == ElectionState.ENDED, "Election is not ended");
        uint winnerId;
        uint maxVotes = 0;
        for (uint i = 1; i <= candidateCount; i++) {
            if (candidates[i].votes > maxVotes) {
                maxVotes = candidates[i].votes;
                winnerId = i;
            }
        }
        return (candidates[winnerId].name, winnerId, maxVotes);
    }
    
    function delegateVote(address _delegate) external onlyOngoingElection onlyVoter {
        require(_delegate != msg.sender, "Cannot delegate to yourself");
        voters[msg.sender].hasDelegated = true;
        voters[msg.sender].delegate = _delegate;
        emit VoteDelegated(msg.sender, _delegate);
    }
    
    function castVote(uint _candidateId) external onlyOngoingElection onlyVoter {
        candidates[_candidateId].votes++;
        voters[msg.sender].votedFor = _candidateId;
        emit VoteCast(msg.sender, _candidateId);
    }
    
    function endElection() external onlyAdmin onlyOngoingElection {
        electionState = ElectionState.ENDED;
        emit ElectionEnded();
    }
    
    function showElectionResults(uint _candidateId) external view returns (uint, string memory, uint) {
        return (_candidateId, candidates[_candidateId].name, candidates[_candidateId].votes);
    }
    
    function viewVoterProfile(address _voter) external view returns (string memory, uint, bool) {
        return (voters[_voter].name, voters[_voter].votedFor, voters[_voter].hasDelegated);
    }
}
