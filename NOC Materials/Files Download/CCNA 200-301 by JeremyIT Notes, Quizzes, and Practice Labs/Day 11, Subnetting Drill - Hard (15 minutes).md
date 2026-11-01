Subnetting drill: hard
No answer key while you drill. Explanations come at the end.

Session complete
35/40

88% correct · best streak 16 · average 22 s per question.

That pace is a good target for exam-style subnetting.

Question type	Correct
Longest shared prefix	1/2
Route summarization	4/6
Broadcast address	2/3
Usable host count	3/4
VLSM allocation		3/3
First usable host	3/3
Last usable host	3/3
Previous subnet		2/2
Next subnet			2/2
Nth subnet			3/3
Wildcard mask		2/2
Valid host check	2/2
Network address		1/1
Mask for N hosts	2/2
Subnet count		1/1
Same subnet check	1/1

Questions to review

What is the smallest single summary route (network/prefix) that covers all of these networks? 94.28.124.128/26 94.28.124.192/26 94.28.125.0/26 94.28.125.64/26
You: 94.28.124.0/22 · Answer: 94.28.124.0/23
The lowest address is 94.28.124.128 and the highest is 94.28.125.127. They share their first 23 bits, so the summary is 94.28.124.0/23.


What is the smallest single summary route (network/prefix) that covers all of these networks? 36.213.32.0/22 36.213.36.0/22 36.213.44.0/22
You: 36.213.32.0/18 · Answer: 36.213.32.0/20
The lowest address is 36.213.32.0 and the highest is 36.213.47.255. They share their first 20 bits, so the summary is 36.213.32.0/20.


How many usable host addresses are on a network with subnet mask 255.255.224.0
You: 286 · Answer: 8,190
32 − 19 = 13 host bits. 2^13 − 2 = 8,190.


What is the longest prefix length that puts these two addresses in the same subnet? 54.146.237.201 54.146.237.221
You: /29 · Answer: /27
Compare the two addresses bit by bit. The first 27 bits match and bit 28 is the first difference, so /27 is the longest prefix that still contains both.


What is the broadcast address of 147.236.240.114/14
You: 147.236.243.255 · Answer: 147.239.255.255
The network is 147.236.0.0 with 262,144 addresses. Set all 18 host bits to 1 and you get 147.239.255.255.