# Earthlings in the Ether

![image](https://github.com/westonnelson/astroship/assets/29180454/8f646b39-6c66-47a7-99a5-5545d57473bd)

## Digitizing Life, One NFT at a Time


# Need to finalize this aspect as the final component before release: here is the smart contract logic for enabling holders of the edition drop to mint the dNFT for a reduced cost if purchased in $NFTE tokens. 

import { ThirdwebSDK } from "@thirdweb-dev/sdk";
import fs from "fs";
import path from "path";

(async () => {
  const sdk = new ThirdwebSDK(“arbitrum);
  // replace the addresses and amount with your own
  const collectionAddress = "0x73AFA57FEC32fE958f1b5E139cDd8D9306E65a6d";
  const tokenAddress = "0xB261104A83887aE92392Fb5CE5899fCFe5481456";
  const tokenAmount = 25000;

  const contract = await sdk.getContract(collectionAddress);

  if (!contract) {
    return console.log("Contract not found");
  }

  // getting all the NFTs of the collection
  const nfts = await contract.erc721.getAll();

  if (!nfts) {
    return console.log("No NFTs found");
  }

  // creating a new array of addresses
  const csv = nfts.reduce((acc, nft) => {
    const address = nft.owner;
    const quantity = acc[address] ? acc[address] + 1 : 1;
    return { ...acc, [address]: quantity };
  }, {});

  // filtering the addressees
  const filteredCsv = Object.keys(csv).reduce((acc, key) => {
    if (key !== "0x0000000000000000000000000000000000000000") {
      return {
        ...acc,
        [key]: csv[key],
      };
    }
    return acc;
  }, {});

  // writing the addresses to a csv file
  const csvString =
    "address,maxClaimable,price,currencyAddress\r" +
    Object.entries(filteredCsv)
      .map(
        ([address, quantity]) =>
          `${address},${quantity},${tokenAmount},${tokenAddress}`
      )
      .join("\r");

  fs.writeFileSync(path.join(path.dirname("."), "snapshot.csv"), csvString);
  console.log("Generated snapshot.csv");
})();
