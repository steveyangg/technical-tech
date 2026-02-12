Provide your CLI command here:

jq -r 'select(.symbol=="TSLA" and .side=="sell") | .order_id' ./transaction-log.txt \
| xargs -I{} curl -s "https://example.com/api/{}" >> ./output.txt

-> This command first reads the transaction log file and filters out only the records where the stock is TSLA and the action is “sell.” From those records, it takes the order IDs and uses them one by one to send HTTP GET requests to the API. Each request calls https://example.com/api/<order_id> for the matching orders. Finally, all the responses from the API are saved into output.txt, so you end up with the results for every TSLA sell order in one file, done automatically in a single line.