# Narrative-alignment-Canopy-style-
Canopy-style L1s redefine how blockchains are launched. Instead of heavy infrastructure upfront, builders deploy L1s like smart contracts with a single click. Chains start virtual on shared resources and can become fully sovereign anytime, aligning experimentation speed with long-term independence.

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title CanopyVirtualL1
 * @author Canopy-style example
 * @notice 1-click Virtual L1 launcher inspired by Canopy Network.
 *
 * Narrative:
 * - Launch an L1 like deploying a smart contract
 * - Start virtual, go sovereign when ready
 * - Minimal on-chain footprint, maximum flexibility
 */
contract CanopyVirtualL1 {
    /*//////////////////////////////////////////////////////////////
                                ERRORS
    //////////////////////////////////////////////////////////////*/
    error NotOwner();
    error InvalidMetadata();
    error L1NotFound();

    /*//////////////////////////////////////////////////////////////
                                EVENTS
    //////////////////////////////////////////////////////////////*/
    event VirtualL1Launched(
        uint256 indexed l1Id,
        address indexed owner,
        string metadataURI
    );

    event VirtualL1UpgradedToSovereign(
        uint256 indexed l1Id
    );

    event MetadataUpdated(
        uint256 indexed l1Id,
        string metadataURI
    );

    /*//////////////////////////////////////////////////////////////
                                STRUCTS
    //////////////////////////////////////////////////////////////*/
    struct VirtualL1 {
        address owner;
        string metadataURI; // Canopy runtime / validator / chain config
        uint64 launchedAt;
        bool sovereign;     // false = virtual, true = independent L1
        bool exists;
    }

    /*//////////////////////////////////////////////////////////////
                                STORAGE
    //////////////////////////////////////////////////////////////*/
    uint256 public totalL1s;
    mapping(uint256 => VirtualL1) private l1s;

    /*//////////////////////////////////////////////////////////////
                                MODIFIERS
    //////////////////////////////////////////////////////////////*/
    modifier onlyOwner(uint256 l1Id) {
        if (!l1s[l1Id].exists) revert L1NotFound();
        if (l1s[l1Id].owner != msg.sender) revert NotOwner();
        _;
    }

    /*//////////////////////////////////////////////////////////////
                            1-CLICK LAUNCH
    //////////////////////////////////////////////////////////////*/

    /**
     * @notice Launch a Virtual L1 in one transaction
     * @dev Equivalent to "deploying an L1 like a smart contract"
     * @param metadataURI Off-chain config (IPFS / Arweave / HTTPS)
     */
    function launchVirtualL1(
        string calldata metadataURI
    ) external returns (uint256 l1Id) {
        if (bytes(metadataURI).length == 0) revert InvalidMetadata();

        unchecked {
            ++totalL1s;
        }

        l1Id = totalL1s;

        l1s[l1Id] = VirtualL1({
            owner: msg.sender,
            metadataURI: metadataURI,
            launchedAt: uint64(block.timestamp),
            sovereign: false,
            exists: true
        });

        emit VirtualL1Launched(l1Id, msg.sender, metadataURI);
    }

    /*//////////////////////////////////////////////////////////////
                        GO SOVEREIGN (ANYTIME)
    //////////////////////////////////////////////////////////////*/

    /**
     * @notice Upgrade a virtual L1 into a sovereign L1
     * @dev Signals independence from shared execution / settlement
     */
    function goSovereign(uint256 l1Id) external onlyOwner(l1Id) {
        l1s[l1Id].sovereign = true;
        emit VirtualL1UpgradedToSovereign(l1Id);
    }

    /*//////////////////////////////////////////////////////////////
                        CONFIG MANAGEMENT
    //////////////////////////////////////////////////////////////*/

    function updateMetadata(
        uint256 l1Id,
        string calldata newMetadataURI
    ) external onlyOwner(l1Id) {
        if (bytes(newMetadataURI).length == 0) revert InvalidMetadata();
        l1s[l1Id].metadataURI = newMetadataURI;
        emit MetadataUpdated(l1Id, newMetadataURI);
    }

    /*//////////////////////////////////////////////////////////////
                                VIEWS
    //////////////////////////////////////////////////////////////*/

    function getL1(uint256 l1Id)
        external
        view
        returns (
            address owner,
            string memory metadataURI,
            uint64 launchedAt,
            bool sovereign
        )
    {
        if (!l1s[l1Id].exists) revert L1NotFound();
        VirtualL1 memory l1 = l1s[l1Id];
        return (l1.owner, l1.metadataURI, l1.launchedAt, l1.sovereign);
    }
}
