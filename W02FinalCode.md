## Broadcast the mined block chain

```java.js
//Blockchain data structure

//0. Call library js-sha256 (for Hash)
const sha256 = require('js-sha256');

//1. 'Constructor' function...data object 
function Blockchain() {
    //1.1 Initialize the chain to an empty array.
    this.chain = []; //We will store all of our blocks here. 
    //1.2 Initialize the transactions empty
    this.newTransactions = []; //Hold all the new transactions before they are "mined" into a block 
    //1.3 Create your first week block:  createNewBlock(nonce,'string value',"string value too")
    this.createNewBlock(100,'0','0');
};

//2. Blockchain code structure. 
//"Each block contains a cryptographic hash of the previous block, a timestamp, and transaction data"
Blockchain.prototype.createNewBlock = function(nonce, previousBlockHash, hash) {
    //2.1. Create a new block, or the first one.
    const newBlock = {
        //2.1.1. Index: returns the position of the first occurrence of specified character(s) in a string.
        index: this.chain.length + 1,
        //2.1.2. Insert timestamp as one of the unique identifiers.
        timestamp: Date.now(),
        //2.1.3. Space for the transactions.
        transactions: this.newTransactions,
        //2.1.4. In order to find the hash, we could define the nonce (an arbitrary number that can be used just once in a cryptographic communication, wiki) 
        //Nonce: number only used once (2, 10, 123). This is the PROOF that we actually created a legit block.
        nonce: nonce,
        //2.1.5. Line for the new Hash (is any function that can be used to map data of arbitrary size to fixed-size values, class notes).         
        //Hash: this data from our new block.
        hash: hash,
        //2.1.6. Space for the prevoius Hash. Each block also references a previous block, known as the parent block, through the “previous block hash” field in the block header, https://www.oreilly.com  
        //Previous Hash: data from our current block hashed into a string.
        previousBlockHash: previousBlockHash  
    };
    //2.2. Clears out any newTransactions.
    this.newTransactions = [];
    //2.3. Add the newBlock the the chain.
    this.chain.push(newBlock);
    //2.4. The return statement stops the execution of a function and returns a value from that function.
    return newBlock;
}

//3. Define Blockchain.prototype
Blockchain.prototype.getLastBlock = function() {
    return this.chain[this.chain.length - 1];
}

//4. Define prototype parameters for the transaction(amount,sender, recipient)
Blockchain.prototype.createNewTransaction = function(amount, sender, recipient) {
    //4.1. Create a newTransaction object
    const newTransaction = {
        amount: amount,
        sender: sender,
        recipient: recipient
    };
    //4.2. Add the new transaction to the newTx data area
    this.newTransactions.push(newTransaction);
    //4.3. Get the index of the last block of our chain plus one, in order to create a new block. 
    return this.getLastBlock()['index'] + 1;
}

//5. Create a Hash function (previousHash, currentblock, nonce)
//What does this code does? Takes the blockData and shares hash as the final result
//Hash example, sha256: "cat" ==> 77af778b51abd4a3c51c5ddd97204a9c3ae614ebccb75a606c3b6865aed6744e
Blockchain.prototype.hashBlock = function(previousBlockHash, currentBlockData, nonce) {
    //5.1. Smoosh / convert all of our 3 parameters into one long string
    const dataAsString = previousBlockHash + nonce.toString() + JSON.stringify(currentBlockData);
    //5.2. Move the string to our new js-sha256 library/package/dependency 
    const hash = sha256(dataAsString);
    //5.3. The return statement stops the execution of a function and returns a the Hash value.
    return hash;
    }


//6. Create Proof of Work, in this case we do not define the nonce, we find it, so... we only work with the previous hash and the new or actual one.
//Difficulty level:the Hash has to start with 000. Increment nonce & run the hash until hash satisfies the difficulty level.
Blockchain.prototype.proofOfWork = function(previousBlockHash, currentBlockData) {  
    //6.1. Define that our loop or runs, start at zero. 'let' is a java instruction that makes the variable change per loop.
    let nonce = 0;
    //6.2. Create the Hash running from zero too.
    let hash = this.hashBlock(previousBlockHash, currentBlockData, nonce);
    //6.3. Execute the code {while the first four digits of the hash are NOT 000} while this condition is true
    while (hash.substring(0, 4) !== '0000') {
        nonce++; // nonce = nonce + 1... 0 + 1 = 1. 1+1 = 2... 
        hash = this.hashBlock(previousBlockHash, currentBlockData, nonce);      
    }

    //6.3. The return statement stops the execution of a function and returns a the Nonce value.
    return nonce;
};

//7. Validate your blockchain
Blockchain.prototype.chainIsValid = function(blockchain) {

    let validChain = true;

    // console.log('how we doing?', validChain);
    // console.log('what/s our bc length?', blockchain.length)

    for (var i = 1; i < blockchain.length; i++) {
        // console.log(i);
        const currentBlock = blockchain[i];
        const prevBlock = blockchain[i - 1];
        const blockHash =
            this.hashBlock(prevBlock['hash'], {
                    transactions: currentBlock['transactions'],
                    index: currentBlock['index']
                },
                currentBlock['nonce']
            );

        if (blockHash.substring(0, 4) != "0000") validChain = false;
        // console.log(blockHash.substring(0, 4));
        // console.log('got 0000s?', validChain);

        if (currentBlock['previousBlockHash'] !== prevBlock['hash']) validChain = false; //chain not valid

        // console.log('got matched hashes to last one?', currentBlock['previousBlockHash'], prevBlock['hash'], validChain);

        // console.log("previousBlockHash==>", prevBlock['hash']);
        // console.log("currentBlockHash ==>", currentBlock['hash']);


    };
    const genesisBlock = blockchain[0];
    const correctNonce = genesisBlock['nonce'] === 100;
    const correctPreviousHash = genesisBlock['previousBlockHash'] === '0';
    const correctHash = genesisBlock['hash'] === '0';
    const correctTransactions = genesisBlock['transactions'].length === 0;

    // console.log(correctNonce, correctPreviousHash, correctHash, correctTransactions);
    // console.log("genesis block hash", genesisBlock['hash'])

    if (!correctNonce || !correctPreviousHash || !correctHash || !correctTransactions) validChain = false;


    return validChain; //true if valid, false if not valid 

}

//8. After that, we do need to keep only the correct ones, the others don't need to be returned. 
Blockchain.prototype.getBlock = function(blockHash) {
    let correctBlock = null;
    this.chain.forEach(block => {
        if (block.hash === blockHash) correctBlock = block;
    });
    return correctBlock;
}

Blockchain.prototype.getTransaction = function(transactionId) {
    let correctTransaction = null;
    let correctBlock = null; //cuz we want to know which block the t/x is in too 

    //loop through every block 
    this.chain.forEach(block => {
        //now loop through each t/x in that block
        block.transactions.forEach(transaction => {
            if (transaction.transactionId === transactionId) {
                correctTransaction = transaction;
                correctBlock = block;
            }
        });
    });
    return {
        transaction: correctTransaction,
        block: correctBlock
    }
}


//9. And finally, see if the transaction has moved into the chain. 
Blockchain.prototype.getAddressData = function(address) {
    //we take in an address, so we have to collect all the t/x into a single array
    const addressTransactions = [];
    //new we cycle through all t/x and look in both sender & receiver to match the passed in address
    this.chain.forEach(block => {
        //now cycle through all t/x per block
        block.transactions.forEach(transaction => {
            //now we have access to every single t/x and we test each for sender or recipient
            if (transaction.sender === address || transaction.recipient === address) addressTransactions.push(transaction); //add it to our array if it matches
        });
    });
    //upon commpletion, we've loaded up the array. Now we have to figure out the balance is for this address
    let balance = 0; //would you do this for you bank? 
    addressTransactions.forEach(transaction => {
        if (transaction.recipient === address) balance += transaction.amount;
        else if (transaction.sender === address) balance -= transaction.amount;
    });

    return {
        addressTransactions: addressTransactions,
        addressBalance: balance
    };

}

module.exports = Blockchain;

```
