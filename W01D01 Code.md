## Blockchain structure
`Blockchain data structure`

```java.js
  //'Constructor' function...data object 
	function Blockchain() {
	    this.chain = []; //Initialize the chain to an empty array. We will store all of our blocks here. 
	    this.newTransactions = []; //Hold all the new t/x before they are "mined" into a block 
	};
	Blockchain.prototype.createNewBlock = function(nonce, previousBlockHash, hash) {
	    //all of these terms will  soon be revealed
	    const newBlock = {
	        index: this.chain.length + 1,
	        //what block is this in our chain. first, or 1000th? 
	        timestamp: Date.now(),
	        transactions: this.newTransactions,
	        //All of the t/x in this block
	        nonce: nonce,
	        //Nonce: number only used once (2, 10, 123). This is the PROOF that we actually created a legit block.
	        hash: hash,
	        //Hash: this data from our new block.
	        previousBlockHash: previousBlockHash
	        //data from our current block hashed into a string
	    };
	    this.newTransactions = [];
	    //clears out any newTransactions
	    this.chain.push(newBlock);
	    //add the newBlock the the chain 
	    return newBlock;
	}
	//First Blockchain.prototype
	Blockchain.prototype.getLastBlock = function() {
	    return this.chain[this.chain.length - 1];
	}
	Blockchain.prototype.createNewTransaction = function(amount, sender, recipient) {
	    //1. Create a newTransaction object
	    const newTransaction = {
	        amount: amount,
	        sender: sender,
	        recipient: recipient
	    };
	    //2. Add the newTx to the newTx data area
	    this.newTransactions.push(newTransaction);
	    //3. Get the index of the last block of our chain plus one, for a new block. 
	    return this.getLastBlock()['index'] + 1;
	}
module.exports = Blockchain;
```
![image](https://user-images.githubusercontent.com/88910721/130320908-7e946afc-7c32-4df5-917f-e4156f24d825.png)

*Notice that in this code we are not printing anything, this would need the instruction:*
>console.log()

### Blockchain test code
	`Blockchain testing mode`

```java.js
// "Anybody can code,just write code that anyone can understand".

//1. Call the Blockchain code
const Blockchain = require('./blockchain.js');

//2. Name or create our "brand" or instance for our data structure (modules)
const rachcoin = new Blockchain(); 

//3. Define console message, shows the object inside ()
console.log(rachcoin);

//4. Create your blocks: name.createNewBlock(nonce, previousBlockHash, hash)
rachcoin.createNewBlock(113277545, 'Driscolls_Services', 'Driscolls_Operations');
rachcoin.createNewBlock(107718716, 'Bemis_Hold', 'Bemis_Active');


//5. Create your transactions: name.createNewTransaction(ammount, sender, receiver)
rachcoin.createNewTransaction(500, 'SENDSERVICES', 'RECEIVEOPERATIONS');
rachcoin.createNewTransaction(100, 'SENDHOLD', 'RECEIVEACTIVE');

//6. Play with the code
//View the entire Blockchain()
console.log(rachcoin);
//Show just one Block
console.log(rachcoin.chain[0].nonce);
//Show just one transaction
console.log(rachcoin.newTransactions[0]);
```
![TestW02D02](https://user-images.githubusercontent.com/88910721/130321293-44089b12-81da-4ae3-9b14-7331aee0f964.png)
![node devtestW02D02](https://user-images.githubusercontent.com/88910721/130321353-fb9d9987-cc74-4654-ab0d-3cdc0020a572.png)

