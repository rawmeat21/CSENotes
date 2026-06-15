The Domain Name System (DNS) is the phonebook of the Internet. 

Humans access information online through [domain names](https://www.cloudflare.com/learning/dns/glossary/what-is-a-domain-name/), like nytimes.com or espn.com. 

Web browsers interact through [Internet Protocol (IP)](https://www.cloudflare.com/learning/network-layer/internet-protocol/) addresses. 

DNS translates domain names to [IP addresses](https://www.cloudflare.com/learning/dns/glossary/what-is-my-ip-address/) so browsers can load Internet resources.

Each device connected to the Internet has a unique IP address which other machines use to find the device. DNS servers eliminate the need for humans to memorize IP addresses.

It translates something like:

**google.com → 142.250.xxx.xxx** This process is called **name resolution**.

Why name resolution exists? There are 2 reasons:

- **IPs can change:** Servers move or scale, but the name (**google.com**) stays the same.
- **Flexibility + Human Readability:** We need a system that humans can actually remember.

![[Pasted image 20260615123748.png]]

### So, let’s say you type your favorite domain into the browser...

1. The browser will first look into its **cache memory** to see if it already has the IP address. If you’ve visited that website before, the browser usually stores the IP to save time.
2. If the browser doesn’t have it, it checks your computer’s Operating System cache next.
3. Otherwise, the browser asks the **Recursive DNS Resolver** (usually provided by your ISP). This resolver is the one that does the heavy lifting, visiting different DNS servers while the browser sits back and waits.
4. First, the Recursive Resolver (RR) asks the **Root Server**. These servers are at the very top of the DNS hierarchy.( There are 13 sets of these root server addresses, strategically placed around the world and operated by 12 different organizations). (It’s not that there are only 13 physical servers, there are actually thousands of servers globally that act as copies of these 13 sets to handle the massive traffic).
5. When the Root Server receives the query, it directs the RR to the **TLD (Top-Level Domain) Server**. These servers store information for top level domains like `.com`, `.in`, `.org`, or `.net`
6. The TLD server directs our RR to the **Authoritative Name Server** (e.g., `ns1.cloudflare.com`). This is the final stop. This server knows everything about the domain, including its IP address, because it has the final authority over that domain's records.
7. The RR asks, _“Please, tell me the IP of example.com,”_ and the Authoritative server finally provides it usually in the form of a **Record**
8. Now, the RR has the IP, It saves the address in its own **cache**, hands it back to the browser, and the browser stores it in _its_ cache too.



## What is “dig” command ?

**Dig** stands for **Domain Information Groper**. It is essentially a DNS debugging and inspection tool that lets you peek under the hood of everything we just talked about.

### What `dig` is used for ?

- Debugging DNS issues
- Understanding **which DNS server answers what**
- Learning DNS resolution flow
- Checking records: `A`, `NS`, `MX`, `TXT`


If you run `dig google.com`,

**It shows you:**

- **Which server replied:** (Usually the **Recursive Resolver** we talked about earlier).
- **What record type
- **TTL (Time to Live):** This is the cache time, basically how long the **Recursive Resolver** or **browser** should remember this IP before asking again.
- **Final IPs:** The actual address of the server.


## DNS Record Types

In the DNS database, we have a **zone file**, and this file contains the records.

Some IMP types:

#### A and AAAA Records

The **A record** is the most common DNS record. It resolves a domain name to an **IPv4 address**. As you can guess, the browser actually needs the A record because that is where the IP address lives.

Similar to the A record, we have the **AAAA record**. This works just like an A record but the AAAA record resolves the domain name to an **IPv6 address**.

#### CNAME Record

Stands for **Canonical Name** record. You can think of this as an “alias” for another domain. Instead of pointing a domain to an IP address (which is what the A record does), a CNAME record points one domain name to another domain name.

Essentially, you can imagine it as a forwarding address. When the resolver sees a CNAME, it makes another DNS query for that new domain to eventually retrieve the A record.

Ex- Pointing `www.google.com` or `mail.google.com` to the root `google.com`.

#### MX Record

The **MX record** stands for **Mail Exchanger**. It is used for emails. The MX record points to the specific server where emails should be delivered for that domain name.

For example, if you are sending an email to `guy@example.com`, the **MTA (Mail Transfer Agent)** will query the MX records for `example.com` to find the correct email server. The name server might respond with something like `mail2.example.com` this is the destination where the mail actually lands.

#### NS Record

The **NS record** stands for **Name Server** record. This record provides the **Authoritative Name Server** for the domain.

For example, if you buy a domain from Hostinger but want to use Cloudflare for your DNS, you would set your NS records to something like `ns1.cloudflare.com`. This tells the internet: "This is the server that holds all the A records and other information for my domain."

#### TXT Record

Finally, we have the **TXT (Text) record**. You can see this as a “note” attached to your domain that other servers can read to verify information or get instructions. It is mainly used for:

- **Verification:** Proving you own a domain (e.g. for Google Search Console).
- **Security:** Email security protocols.


References:

https://www.cloudflare.com/learning/dns/what-is-dns/
https://medium.com/@pulkit8129/how-dns-resolution-works-a3c5fd46440f
https://medium.com/@pulkit8129/dns-record-types-explained-d2f09d38dbf9


