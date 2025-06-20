// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title Academic Credential Verification
 * @dev Smart contract for issuing and verifying academic credentials on blockchain
 * @author Academic Credential Verification Team
 */
contract Project {
    
    // Structure to store credential information
    struct Credential {
        string studentName;
        string institutionName;
        string degree;
        string fieldOfStudy;
        uint256 graduationYear;
        string ipfsHash; // IPFS hash for storing detailed certificate data
        bool isValid;
        uint256 issuedDate;
        address issuer;
    }
    
    // Mapping from credential ID to credential details
    mapping(bytes32 => Credential) public credentials;
    
    // Mapping to track authorized institutions
    mapping(address => bool) public authorizedInstitutions;
    
    // Mapping to track institution names
    mapping(address => string) public institutionNames;
    
    // Owner of the contract (can authorize institutions)
    address public owner;
    
    // Events
    event CredentialIssued(
        bytes32 indexed credentialId,
        string studentName,
        string institutionName,
        string degree,
        address indexed issuer
    );
    
    event InstitutionAuthorized(
        address indexed institution,
        string institutionName
    );
    
    event CredentialRevoked(
        bytes32 indexed credentialId,
        address indexed revoker
    );
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can perform this action");
        _;
    }
    
    modifier onlyAuthorizedInstitution() {
        require(authorizedInstitutions[msg.sender], "Only authorized institutions can issue credentials");
        _;
    }
    
    modifier credentialExists(bytes32 _credentialId) {
        require(credentials[_credentialId].issuedDate != 0, "Credential does not exist");
        _;
    }
    
    constructor() {
        owner = msg.sender;
    }
    
    /**
     * @dev Core Function 1: Issue a new academic credential
     * @param _studentName Name of the student
     * @param _degree Type of degree (Bachelor's, Master's, PhD, etc.)
     * @param _fieldOfStudy Field of study
     * @param _graduationYear Year of graduation
     * @param _ipfsHash IPFS hash containing detailed certificate information
     * @return credentialId Unique identifier for the issued credential
     */
    function issueCredential(
        string memory _studentName,
        string memory _degree,
        string memory _fieldOfStudy,
        uint256 _graduationYear,
        string memory _ipfsHash
    ) external onlyAuthorizedInstitution returns (bytes32) {
        
        require(bytes(_studentName).length > 0, "Student name cannot be empty");
        require(bytes(_degree).length > 0, "Degree cannot be empty");
        require(bytes(_fieldOfStudy).length > 0, "Field of study cannot be empty");
        require(_graduationYear > 1900 && _graduationYear <= block.timestamp / 365 days + 1970, "Invalid graduation year");
        require(bytes(_ipfsHash).length > 0, "IPFS hash cannot be empty");
        
        // Generate unique credential ID
        bytes32 credentialId = keccak256(
            abi.encodePacked(
                _studentName,
                _degree,
                _fieldOfStudy,
                _graduationYear,
                msg.sender,
                block.timestamp
            )
        );
        
        // Ensure credential doesn't already exist
        require(credentials[credentialId].issuedDate == 0, "Credential already exists");
        
        // Create and store credential
        credentials[credentialId] = Credential({
            studentName: _studentName,
            institutionName: institutionNames[msg.sender],
            degree: _degree,
            fieldOfStudy: _fieldOfStudy,
            graduationYear: _graduationYear,
            ipfsHash: _ipfsHash,
            isValid: true,
            issuedDate: block.timestamp,
            issuer: msg.sender
        });
        
        emit CredentialIssued(
            credentialId,
            _studentName,
            institutionNames[msg.sender],
            _degree,
            msg.sender
        );
        
        return credentialId;
    }
    
    /**
     * @dev Core Function 2: Verify the authenticity of a credential
     * @param _credentialId Unique identifier of the credential to verify
     * @return isValid Whether the credential is valid
     * @return studentName Name of the student
     * @return institutionName Name of the issuing institution
     * @return degree Type of degree
     * @return fieldOfStudy Field of study
     * @return graduationYear Year of graduation
     * @return ipfsHash IPFS hash for detailed information
     */
    function verifyCredential(bytes32 _credentialId) 
        external 
        view 
        credentialExists(_credentialId)
        returns (
            bool isValid,
            string memory studentName,
            string memory institutionName,
            string memory degree,
            string memory fieldOfStudy,
            uint256 graduationYear,
            string memory ipfsHash
        ) 
    {
        Credential memory cred = credentials[_credentialId];
        
        return (
            cred.isValid,
            cred.studentName,
            cred.institutionName,
            cred.degree,
            cred.fieldOfStudy,
            cred.graduationYear,
            cred.ipfsHash
        );
    }
    
    /**
     * @dev Core Function 3: Authorize an educational institution to issue credentials
     * @param _institution Address of the institution to authorize
     * @param _institutionName Name of the institution
     */
    function authorizeInstitution(
        address _institution,
        string memory _institutionName
    ) external onlyOwner {
        require(_institution != address(0), "Invalid institution address");
        require(bytes(_institutionName).length > 0, "Institution name cannot be empty");
        require(!authorizedInstitutions[_institution], "Institution already authorized");
        
        authorizedInstitutions[_institution] = true;
        institutionNames[_institution] = _institutionName;
        
        emit InstitutionAuthorized(_institution, _institutionName);
    }
    
    /**
     * @dev Revoke a credential (can only be done by the issuing institution)
     * @param _credentialId Unique identifier of the credential to revoke
     */
    function revokeCredential(bytes32 _credentialId) 
        external 
        credentialExists(_credentialId)
    {
        require(
            credentials[_credentialId].issuer == msg.sender || msg.sender == owner,
            "Only the issuing institution or owner can revoke credentials"
        );
        require(credentials[_credentialId].isValid, "Credential is already revoked");
        
        credentials[_credentialId].isValid = false;
        
        emit CredentialRevoked(_credentialId, msg.sender);
    }
    
    /**
     * @dev Get the total number of credentials issued by an institution
     * @param _institution Address of the institution
     * @return count Number of credentials issued (Note: This is a simplified implementation)
     */
    function getInstitutionInfo(address _institution) 
        external 
        view 
        returns (
            bool isAuthorized,
            string memory institutionName
        ) 
    {
        return (
            authorizedInstitutions[_institution],
            institutionNames[_institution]
        );
    }
    
    /**
     * @dev Remove authorization from an institution
     * @param _institution Address of the institution to deauthorize
     */
    function deauthorizeInstitution(address _institution) external onlyOwner {
        require(authorizedInstitutions[_institution], "Institution is not authorized");
        
        authorizedInstitutions[_institution] = false;
        delete institutionNames[_institution];
    }
    
    /**
     * @dev Transfer ownership of the contract
     * @param _newOwner Address of the new owner
     */
    function transferOwnership(address _newOwner) external onlyOwner {
        require(_newOwner != address(0), "New owner cannot be zero address");
        owner = _newOwner;
    }
}
