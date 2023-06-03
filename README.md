# nftroyalities.sol

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract MyNFT is ERC721 {
    using SafeMath for uint256;

    // Royalty fee percentage for creator
    uint256 public royaltyFee;

    // Mapping of token ID to creator address
    mapping(uint256 => address) public creators;

    constructor(string memory _name, string memory _symbol)
        ERC721(_name, _symbol)
    {
        royaltyFee = 10; // Set royalty fee to 10% by default
    }

    // Function to set royalty fee percentage
    function setRoyaltyFee(uint256 _fee) external onlyOwner {
        require(_fee <= 100, "MyNFT: Royalty fee cannot exceed 100%");
        royaltyFee = _fee;
    }

    // Function to create new NFT
    function mint(address _to, uint256 _tokenId) external onlyOwner {
        _mint(_to, _tokenId);
        creators[_tokenId] = msg.sender; // Set creator of NFT
    }

    // Function to transfer NFT ownership with royalties
    function safeTransferFromWithRoyalty(
        address _from,
        address _to,
        uint256 _tokenId
    ) public payable {
        address creator = creators[_tokenId];
        uint256 royaltyAmount = msg.value.mul(royaltyFee).div(100);
        uint256 transferAmount = msg.value.sub(royaltyAmount);

        require(
            creator != address(0),
            "MyNFT: Token does not exist or creator not set"
        );
        require(
            ownerOf(_tokenId) == _from,
            "MyNFT: Sender is not owner of token"
        );
        require(_to != address(0), "MyNFT: Invalid recipient address");

        // Transfer ownership of NFT
        safeTransferFrom(_from, _to, _tokenId);

        // Transfer royalty fee to creator
        payable(creator).transfer(royaltyAmount);

        // Transfer remaining amount to seller
        payable(_from).transfer(transferAmount);
    }

    // Function to withdraw royalty earnings
    function withdrawRoyaltyEarnings() public {
        uint256 balance = address(this).balance;
        require(balance > 0, "MyNFT: No earnings available to withdraw");

        payable(msg.sender).transfer(balance);
    }
}
