## Index
- Key concepts
- New library: npm install js-sha256 (admin rights needed)
- Hash Code
- Proof of work Code
- Writing functions Tips

### Hash

`blockchainHash.js`
  
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

//5. Creating a Hash function (previousHash, currentblock, nonce)
Blockchain.prototype.hashBlock = function(previousBlockHash, currentBlockData, nonce) {
    //5.1. Smoosh / convert all of our 3 parameters into one long string
    const dataAsString = previousBlockHash + nonce.toString() + JSON.stringify(currentBlockData);
    //5.2. Move the string to our new js-sha256 library/package/dependency 
    const hash = sha256(dataAsString);
    //5.3. The return statement stops the execution of a function and returns a the Hash value.
    return hash;
    }
//What does this code does? Takes the blockData and shares hash as the final result
//Hash example, sha256: "cat" ==> 77af778b51abd4a3c51c5ddd97204a9c3ae614ebccb75a606c3b6865aed6744e
module.exports = Blockchain;

```
> New code

![BlockchainHashW01D02](https://user-images.githubusercontent.com/88910721/130329454-a38b7269-ad98-42ab-af01-6b7f5680c5bf.PNG)


*Notice that in this code we are not printing anything, again.*


`testHash.js`

```java.js
// "Anybody can code,just write code that anyone can understand".

//1. Call the Blockchain Hash code
const Blockchain = require('./blockchainHash.js');

//2. Name or create our "brand" or instance for our data structure (modules)
const rachcoin = new Blockchain(); 

//3. Print rachcoin with timestamp + nonce + hash + previous hash
console.log(rachcoin);

//4. Create your blocks: name.createNewBlock(nonce, previousBlockHash, hash)
rachcoin.createNewBlock(113277545, 'Driscolls_Services', 'Driscolls_Operations');
rachcoin.createNewBlock(107718716, 'Bemis_Hold', 'Bemis_Active');


//5. Create your transactions: name.createNewTransaction(ammount, sender, receiver)
rachcoin.createNewTransaction(200000, 'SENDSERVICES', 'RECEIVEOPERATIONS');
rachcoin.createNewTransaction(50000, 'SENDHOLD', 'RECEIVEACTIVE');

//6. Identify the Hash
//6.1. Define the Previous Block Hash we'll be following, rachcoin.previousBlockHash SHA-256 Hash
const previousBlockHash = '8f1288b4a9817dcd091d1f0b09d7531c37bb10760e155f1243d9616b0dbba79c';
//6.2. Create printing structure and parameters of the new block
const currentBlockData = [{
        "Amount": 200000,
        "Sender": "SENDSERVICES",
        "Recipient": "RECEIVEOPERATIONS",
    },
    {
        "amount": 50000,
        "sender": "SENDHOLD",
        "recipient": "RECEIVEACTIVE",
    }
];
const nonce = 100;

//7. View the Hash guided by the previous block hash, de current data and the defined nonce. 
console.log(rachcoin.hashBlock(previousBlockHash, currentBlockData, nonce));

```
> Terminal node dev/testHash.js

![TestHashW01D02](https://user-images.githubusercontent.com/88910721/130329603-5b580e58-ae05-40a5-9da2-05f24565b81d.PNG)

### Proof of work

`blockchainProof.js`
  
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
    while (hash.substring(0, 4) !== '000') {
        nonce++; // nonce = nonce + 1... 0 + 1 = 1. 1+1 = 2... 
        hash = this.hashBlock(previousBlockHash, currentBlockData, nonce);      
    }
    //6.3. The return statement stops the execution of a function and returns a the Nonce value.
    return nonce;
};
module.exports = Blockchain;

```
> New code

![BlockchainProofW01D02](https://user-images.githubusercontent.com/88910721/130330929-c81f1c37-5723-49c3-84e3-9c9571017c2e.PNG)


'testProof.js`

```java.js

// "Anybody can code,just write code that anyone can understand".

//1. Call the Blockchain Proof code
const Blockchain = require('./blockchainProof.js');

//2. Name or create our "brand" or instance for our data structure (modules)
const rachcoin = new Blockchain(); 

//3. Create your blocks: name.createNewBlock(nonce, previousBlockHash, hash)
rachcoin.createNewBlock(113277545, 'Driscolls_Services', 'Driscolls_Operations');
rachcoin.createNewBlock(107718716, 'Bemis_Hold', 'Bemis_Active');

//4. Create your transactions: name.createNewTransaction(ammount, sender, receiver)
rachcoin.createNewTransaction(200000, 'SENDSERVICES', 'RECEIVEOPERATIONS');
rachcoin.createNewTransaction(50000, 'SENDHOLD', 'RECEIVEACTIVE');

//5. Identify the Hash
//5.1. Define the Previous Block Hash we'll be following, rachcoin.previousBlockHash SHA-256 Hash
const previousBlockHash = '8f1288b4a9817dcd091d1f0b09d7531c37bb10760e155f1243d9616b0dbba79c';
//5.2. Create printing structure and parameters of the new block
const currentBlockData = [{
        "Amount": 200000,
        "Sender": "SENDSERVICES",
        "Recipient": "RECEIVEOPERATIONS",
    },
    {
        "amount": 50000,
        "sender": "SENDHOLD",
        "recipient": "RECEIVEACTIVE",
    }
];

//6. Now, we are not going to define the nonce, we are looking for it. 
let nonce = rachcoin.proofOfWork(previousBlockHash, currentBlockData);
//View the noce with some text before
console.log('nonce from Proof of Work : ' + nonce);
//7. View the Hash related to the nonce above
console.log(rachcoin.hashBlock(previousBlockHash, currentBlockData, nonce));
//8. View the blockchain 
console.log(rachcoin);

```
> Terminal node dev/testProof.js
