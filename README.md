# WHOIS-DB
An Offline WHOIS DataBase that normalizes ALL WHOIS Registrar records into a functional SQL DataBase. Since it's primary format is CSV, it is extremely versatile. I designed a "Host Classification" system that can classify records as VPS Hosting, Medical, ISP/Telecomm, Educational, Government, and potentially more. The DB is also extensible to include investigation notes, historical changes & notes, NMAP output, site content from record investigations, and more. Your standard WHOIS information available from any service provider comes at a cost and has legal limitations on what records can be collected. RWHOIS links are not allowed to be followed either so ISP to Customer information is not always present. Their records are also stale since they are not updated in real time and sometimes years out of date.

My WHOIS DB does NOT have those limitations, specifically legality and staleness. My WHOIS DB collected the most current record the day you make the request. Since the information my WHOIS DB can collect is not sold (I don't sell it and if you don't sell it you have the same advantage), you CAN follow the RWHOIS links which can give you more insight. A common example is SBCGlobal. All WHOIS services can only provide information down to the ISP, SBCGlobal however SBCGlobal has RWHOIS links that include the idividual customer who has the leased IP block. This is useful for (D)DoS attacks since you can identify a single IP/CIDR block for a single customer of an ISP that has been compromised and Block them. If you only see the entire CIDR block for SBCGlobal which is all you see with a paid for WHOIS Service, you can't take that type of action and must continue to be attacked by the compromised user. You can still block the IP, but having the additional information give justification since you know it's an owned IP, vs. a rotating IP.

The WHOIS DB's main output is a sortable CSV file with over 1,000 columns of data points it leverages. The cause for so many columns is the diverse, highly inconsistent WHOIS record fields, particularly across the various Registrars. To normalize the data to accomodate a searchable, functionable DB, it extended to this depth for columns. Not all columns are used in every investigation, so you typically focus on the key ones you need but the additional information is there and additional, technical, USEFUL information is always key for investigations.

It does output to JSON, a normalized version, and the original records. There are a ton of features and as time goes on I will document them in a more functional Manual. The code is commented extensively. I leverage the CSV output for Splunk searches to inject WHOIS info into my Security Incident Investigation Reports, and general Splunk searches. I've leveraged it for NMAP return fire (they felt me up so let's return the favor), and collecting a list of web pages to investigate to see how legitimate the company that "felt me up" is.

The WHOIS DB runs from a root path of anywhere you want. With my build, I keep all my scripts in scripts/new_queries but I believe I your scripts are in just scripts which is fine since the path is agnostic from where it runs from. 

There are two scripts that do all the magic. They are 01_new_queries_whois and 02_whois_full_process.
01_new_queries takes a list of IPs you feed it in the n00b_query_prep file. It's a friendly script where it has a variety of prompts to guide you thru running the WHOIS query since there are several options to consider. One key thing it does is a bunch of look ups to see if they already exist, and/or have changed then let you decide how you want to handle the new requests IF you want to re-request what exists as is. Finally it will ask if you want to run through the host classification system which is very helpful to classify the types of hosts you are looking up such as TOR Exit nodes, VPS/Hosting providers, both which are frequent sources of attacks, or perhaps ISP/Telecomm which may not be as easy to block, or perhaps HealthCrae, Education, or Government, which have their own implications. You can extend this classification system yourself as well ;) Once you get thru that and your WHOIS records are saved, then it's time to process them. 

02_whois_full_process takes all the new WHOIS records you've saved and runs them through the normalization process. This is nearly 100% automated in how it runs assuming you have all the directory structures built out. The scripts initial comments explain more about this as well does this write up. This can be very time consuming and one day I do intend to move this to Python or similar so it runs in memory vs. disk operations. The normalization normalizing things like the field titles like INETNUM vs NetNumber (same field value just different name depending on Registrar/ISP/etc.), IP/CIDR block/range calculations, and really just a LOT of things. There's over 800 lines of code and some are exceptionally lengthy one liners ;) 

From the root path of choice, you can just paste this script in or drop it in an executable and run it. It will then create all the paths you need. From there you need to feed your IP list one IP per line with nothing else in the "n00b_queries/n00b_query_list". 

Once you have a list you run the "build_whois_query_list" script and it will randomize the list so you don't beat on a single registrar too hard, then ask you if you want to start querying the list. You can choose no and then go to "n00b_queries" and run the script it created there for you. The script name is always "n00b_query_YYYY_MM_DD" so you can keep an archive of your daily queries. You can expand on that time stamp to have more records if you'd like with the date variables at the top of the script.

The catalogs are useful for tracking the inventory of the system and if you add a time stamp to the end of the command you can keep a historical inventory.

EIKR means "exists in known registrar". That means you have a copy already. This script checks against what exists already and moves files over there if they already exist. It does a diff during this process to also see if the information is new/different. If there is something new, then the new stuff will be in "n00b_queries/ready_db_updated/*" for someone to inspect or autoload if you so choose.

If the query is genuinely a new query then once is runs through all it's processing it will end up in "noob_queries/ready_db" as a CSV file unique to each IP that was queried. This was the most effective way to load into the database and recover from error that I found rather than a single file but a single file is also created in "csv_registrars/all/" so you can build a database with one file and also work with a single CSV tho be warned it gets big and Excel especially on Mac for an assumable reason, doesn't work out so well.

All queries in processing will be in their respective format named paths under "n00b_queries" like csv, json, norm, and loadable (meaning original). Once the files are completed processing, all files are removed from said paths and then moved to their final destination which is "*_registars" where * is csv, json, norm, or the originals which is "registrars".

If something failed in processing it will be in "n00b_queries/not_processed". The typical reason this happens is there was no IP block, you exceeded queries, or a new regiSTrAR is born.

All directories are controlled by variables in the script so you can actually change the names of the paths I've selected below to whatever you want so long as you update the variable at the top of the script. I've learned the hard way of using relative non-variable linked and static paths on random systems.

As said the only binary you should need additional is ipcalc. I recommend PostgreSQL 9.2 or higher with the inet functions (additional libs to install) so you can query from a single IP against the networks.

All registrar entries have a root CIDR block whether it be a range or a real CIDR designated to it.  Usually it is called NetRange or inetnum but I just went with inetnum. Sometimes there are additional ones underneath it in the same query. These are all turned into real CIDR blocks and put into the various inetnum_* columns sequentially as they are seen down the query.

There are some columns for network and host*. Check those out as they compliment the inetnum value such as netmask hostmin hostmax netcount.

All BGP AS are extracted into the originas columns.

All e-mail addressses are extracted and put into 10 unique email_* columns.

All abuse is called abuse and all physical address info is under w_address. 

Countries are extracted and should all be capitalized and set to only a 2 digit code even if it was 3 to comply with RFC/lookup.

A lot of other columns are concat'd such as org stuff etc. You can dig into the script to really understand how all that works. It's the section where you see around 700 nearly repetitious grep insert if not exists.

Beyond that you should be good.


mkdir n00b_queries
mkdir n00b_queries/norm
mkdir n00b_queries/process
mkdir n00b_queries/done
mkdir n00b_queries/json
mkdir n00b_queries/not_processed
mkdir n00b_queries/ready_db_updated
mkdir n00b_queries/not_processed_eikr/norm
mkdir n00b_queries/not_processed_eikr/json
mkdir n00b_queries/not_processed_eikr/loadable
mkdir n00b_queries/not_processed_eikr/csv
mkdir n00b_queries/not_processed_eikr
mkdir n00b_queries/loadable
mkdir n00b_queries/ready_db
mkdir n00b_queries/csv
mkdir csv_registrars
mkdir csv_registrars/RIPE
mkdir csv_registrars/AfriNIC
mkdir csv_registrars/LACNIC
mkdir csv_registrars/all
mkdir csv_registrars/KRNIC
mkdir csv_registrars/Nic.br
mkdir csv_registrars/JPNIC
mkdir csv_registrars/OTHER
mkdir csv_registrars/APNIC
mkdir csv_registrars/ARIN
mkdir csv_registrars/TWNIC
mkdir registrars
mkdir registrars/RIPE
mkdir registrars/AfriNIC
mkdir registrars/LACNIC
mkdir registrars/all
mkdir registrars/KRNIC
mkdir registrars/Nic.br
mkdir registrars/JPNIC
mkdir registrars/OTHER
mkdir registrars/APNIC
mkdir registrars/ARIN
mkdir registrars/TWNICmkdir catalog
mkdir json_registrars
mkdir json_registrars/RIPE
mkdir json_registrars/AfriNIC
mkdir json_registrars/LACNIC
mkdir json_registrars/all
mkdir json_registrars/KRNIC
mkdir json_registrars/Nic.br
mkdir json_registrars/JPNIC
mkdir json_registrars/OTHER
mkdir json_registrars/APNIC
mkdir json_registrars/ARIN
mkdir json_registrars/TWNIC
mkdir normalized_registrars
mkdir normalized_registrars/RIPE
mkdir normalized_registrars/AfriNIC
mkdir normalized_registrars/LACNIC
mkdir normalized_registrars/all
mkdir normalized_registrars/KRNIC
mkdir normalized_registrars/Nic.br
mkdir normalized_registrars/JPNIC
mkdir normalized_registrars/OTHER
mkdir normalized_registrars/APNIC
mkdir normalized_registrars/ARIN
mkdir normalized_registrars/TWNIC
mkdir scripts
mkdir scripts/new_queries
mkdir scripts/ZZZ_admin_scripts
echo "./COMMAND_THAT_BLOWS_UP_ENTIRE_COMPANY.sh"
echo ";)"
