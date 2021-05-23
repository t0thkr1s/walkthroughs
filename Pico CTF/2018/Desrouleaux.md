# Desrouleaux

This walkthrough is going to be a little bit different. This challenge is from picoCTF 2018 game. I don’t intend to do these kinds of CTF walkthroughs, but since I really enjoyed this particular problem I decided to publish my solution.

## The Problem

> Our network administrator is having some trouble handling the tickets for all of our incidents. Can you help him out by answering all the questions?
Connect with nc 2018shell1.picoctf.com 10493

They provided an incidents.json file:

```javascript
{
    "tickets": [
        {
            "ticket_id": 0,
            "timestamp": "2015/06/14 10:04:55",
            "file_hash": "dd8e7c221065e9d5",
            "src_ip": "212.230.105.141",
            "dst_ip": "78.209.73.184"
        },
        {
            "ticket_id": 1,
            "timestamp": "2016/01/13 12:28:22",
            "file_hash": "1080f3370465a1a3",
            "src_ip": "166.37.42.57",
            "dst_ip": "45.168.121.69"
        },
        {
            "ticket_id": 2,
            "timestamp": "2016/04/11 01:56:12",
            "file_hash": "c634819f904c97cf",
            "src_ip": "21.201.106.115",
            "dst_ip": "220.119.53.102"
        },
        {
            "ticket_id": 3,
            "timestamp": "2017/09/28 19:25:11",
            "file_hash": "677be057e63c769f",
            "src_ip": "166.37.42.57",
            "dst_ip": "220.119.53.102"
        },
        {
            "ticket_id": 4,
            "timestamp": "2015/01/04 21:23:45",
            "file_hash": "dd8e7c221065e9d5",
            "src_ip": "21.201.106.115",
            "dst_ip": "62.207.187.94"
        },
        {
            "ticket_id": 5,
            "timestamp": "2017/04/20 00:53:55",
            "file_hash": "5d9930783bfc1fb2",
            "src_ip": "192.52.177.68",
            "dst_ip": "183.129.185.131"
        },
        {
            "ticket_id": 6,
            "timestamp": "2015/10/02 10:05:10",
            "file_hash": "fdf0833a308465e6",
            "src_ip": "192.52.177.68",
            "dst_ip": "62.207.187.94"
        },
        {
            "ticket_id": 7,
            "timestamp": "2016/02/15 05:05:39",
            "file_hash": "ccc699fa499d69ae",
            "src_ip": "21.201.106.115",
            "dst_ip": "50.11.106.46"
        },
        {
            "ticket_id": 8,
            "timestamp": "2016/09/16 03:18:51",
            "file_hash": "4d62f65d434d2d8c",
            "src_ip": "21.201.106.115",
            "dst_ip": "124.14.200.83"
        },
        {
            "ticket_id": 9,
            "timestamp": "2017/11/30 11:52:19",
            "file_hash": "e8f7b667d804f295",
            "src_ip": "231.39.42.54",
            "dst_ip": "124.14.200.83"
        }
    ]
}
```

### Question #1

Alright. The first question was: “What is the most common source IP address? If there is more than one IP address that is the most common, you may give any of the most common ones.”  This one was pretty easy… I went through the JSON file and used the max() function on a set() to get the most common IP address in the list.

### Question #2

Let’s move onto the next question: “How many unique destination IP addresses were targeted by the source IP address 21.201.106.115?” Now, this one was interesting, because if you reconnect you may get a new IP address in the question. I found a regex filter to find the IP address in a string. After this, I had to go through the JSON file. If the source IP was equal to the IP in the question and the destionation IP was not in the unique destination list, then I added the IP to the unique list. The answer is the length of this list.

### Question #3

And here comes the final and most confusion question in the whole game…
“What is the average number of unique destination IP addresses that were sent a file with the same hash? Your answer needs to be correct to 2 decimal places.” I spent almost 2 day on figuring out, what should I do here. I wasn’t familiar with pwntools, but I have seen others using it for these kind of challenges and I gave it a go. Basically, I created an answer variable with an initial value of 0.00. If I got “Incorrect!” as a result, I closed the connection and after 2 sec I incremented the answer variable by 0.01. Obviously, this whole process was inside a big inifite loop.

## Full Source Code

Here is the full source code that I wrote for this challenge.

```python3
#!/usr/bin/env python

import json

from pwn import *


def main():
    with open('incidents.json', 'r')as f:
        json_string = json.load(f)

    answer = 0.00
    while True:

        r = remote('2018shell1.picoctf.com', 10493)

        print r.recvuntil("questions.") + "\n"

        # Question 1

        question_1 = r.recvuntil("ones.").strip()
        print "The first question is: " + question_1

	      source_ip_list = []
        for value in json_string["tickets"]:
	          source_ip_list.append(value["src_ip"])

        most_common_source_ip = max(set(source_ip_list), key=source_ip_list.count)
	      print most_common_source_ip
        r.send(most_common_source_ip + "\n")
        r.recv()
        print "Result: " + r.recvline()
        r.recvline()
        r.recvline()

        # Question 2

        question_2 = r.recvuntil("?")

        print "The second question is: " + question_2

        ip_address = re.findall(r'[0-9]+(?:\.[0-9]+){3}', question_2)

        unique_destination_ip_addresses = []
        for value in json_string['tickets']:
            if (value['src_ip'] == ip_address.__getitem__(0) 
            and value['dst_ip'] not in unique_destination_ip_addresses):
                unique_destination_ip_addresses.append(value['dst_ip'])

        print str(len(unique_destination_ip_addresses))
        r.send(str(len(unique_destination_ip_addresses)) + "\n")
        r.recv()
        print "Result: " + r.recvline()
        r.recvline()
        r.recvline()

        # Question 3

        question_3 = r.recvuntil("places.")

        print("The third question is: " + question_3)

        print "{:.2f}".format(answer)
        r.send("{:.2f}".format(answer) + "\n")
        r.recv()
        result = r.recvline()
        print "Result: " + result

        if result == "Correct!\n":
            print r.recv()
            break
        elif result == "Incorrect!\n":
            r.close()
            print "\nAnswer is incorrect retrying after 2 sec ...\n"
            time.sleep(2)
            answer += 0.01
            continue


if __name__ == '__main__':
    main()
```
