#Deposit function

Allow users to deposit coin into their wallets 

## Scenario
Main scenario when user deposit:
- When user click on deposit, front-end send request contain wallet address to backend
- If this is first time, backend create a address called is quote address based on master key(only one master key for all)
and index(unique for each deposit) and send this address to front-end. And backend save this address to deposit table 
- If user re click on deposit, backend send the existing quote address(taken from deposit table) and pending status to 
front-end
- If user transfer money, nova service will update status of this deposit is waiting confirm, and number of deposit times 
will be count and save to current_block_confirm in  deposit table
- If current_block_confirm enough, update status is success and append this deposit to txt_reciever field(this field is array)
in deposit table
- When user redeposit, back-end will delete old row corresponding to wallet, and create new quote address and new row in 
deposit table
When user deposit:
- When user click on history, front-end send request contain wallet address to backend to get history of deposit times. 
- Backend get txt_reciever field corresponding to this wallet, and send back to front-end 

## api
- Get generate quote address: 
`GET /deposit/generate-address?UserAddr=[Wallet address]`
- Get History:
`GET /deposit/history?UserAddr=[Wallet address]`
- Get schema: 
`GET /deposit/schema`

