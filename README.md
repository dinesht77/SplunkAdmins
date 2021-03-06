## SplunkBase
Also available on SplunkBase as [Alerts for Splunk Admins](https://splunkbase.splunk.com/app/3796/)
You may also be interested in [VersionControl For Splunk](https://splunkbase.splunk.com/app/4355/)

## Introduction
This application accompanies the Splunk conf 2017 presentation "How did you get so big? Tips and tricks for growing your Splunk installation from 50GB/day to 1TB/day"

The overall idea behind this application is to provide a variety of alerts that detect issues or potential issues within the splunk log files and then advise via an alert that this has occurred
This application was built as there were a variety of messages in the Splunk console and logs in Splunk that if acted upon could have prevented an issue within the environment.

The original presentation is available as a [recording](http://conf.splunk.com/files/2017/recordings/howd-you-get-so-big-tips-n-tricks-for-growing-your-splunk-deployment-from-50-gb-per-day-to-1-tb-per-day.mp4) or [PDF](http://conf.splunk.com/files/2017/slides/howd-you-get-so-big-tips-tricks-for-growing-your-splunk-deployment-from-50-gb-day-to-1-tb-day.pdf)
The powerpoint should it be required is available [here](https://github.com/gjanders/splunkconf2017)

There are many potential alerts that might cause an issue so this application has all alerts disabled by default, post-installation once the required macros are configured you can enable the alerts you wish to use and add the required actions

There are also a few dashboards for investigating indexer performance, heavy forwarder queue usage and data model acceleration issues

Please note that the all alerts & dashboards were tested on Linux-based Splunk infrastructure, with AIX, Linux and Windows forwarders

If you are running your Splunk enterprise installation on Windows or have customised your installation directory you will need to customise some of the macros such as `splunkadmins_splunkd_source` to point to the correct splunkd log file location

Also note that this application contains a very large number of alerts which you can use, you may wish to utilise the `allow_skew` in savedsearches.conf to allow the scheduler to balance out the scheduled alerts execution times

## Macros - required configuration
The various saved searches and dashboards use macros within their searches, you will need to update the macros to ensure the searches/dashboards work as expected
To check the contents of the macros in Splunk 7 or newer, use CTRL-SHFT-E within the search window

The macros are listed below, many expect a `host=A OR host=B` item to assist in narrowing down a search while others expect only a single value...note that for `splunk_server` values they are always lower-case and case-sensitive!

`indexerhosts - a host=...` list of your indexers (for example `host=indexer1 OR host=indexer2`)

`heavyforwarderhosts - a host=...` list of your heavy forwarders (for example `host=heavyforwarder1 OR host=heavyforwarder2`)

`searchheadhosts - a host=...` list of your search head(s) (for example `host=searchhead1 OR host=searchhead2`)

`localsearchheadhosts - a host=...` list of your search head(s) within the cluster that these alerts are running on

`splunkenterprisehosts - a host=...` list of any Splunk enterprise instance (for example `host=indexer1 OR host=searchhead1 OR ...`)

`deploymentserverhosts - a host=...` list of deployment server(s) (for example `host=splunkdeploymentserver`)

`licensemasterhost - a host=...` entry for the license master server (for example `host=splunklicensemaster`)

`searchheadsplunkservers - a splunk_server=...` list of any Splunk search head hosts (for example `splunk_server=searchhead*`)

`splunkindexerhostsvalue - a splunk_server=...` list of any Splunk indexer hosts (for example `splunk_server=indexer*`)

`splunkadmins_splunkd_source` - this defaults to `source=*splunkd.log`, for a slight improvement in performance you can make this a specific file such as `/opt/splunk/var/log/splunk/splunkd.log`

`splunkadmins_splunkuf_source` - this defaults to `source=*splunkd.log`, you may wish to narrow down this location if your splunkd logs on universal forwarders have consistent installation directories

`splunkadmins_mongo_source` - this defaults to `source=*mongod.log`, for a slight improvement in performance you can make this a specific file such as `/opt/splunk/var/log/splunk/mongod.log`

`splunkadmins_clustermaster_oshost - a host=...` entry for the cluster master server (for example `host=splunkclustermaster`)

The macros are used in various alerts which you can optionally enable, the alerts will raise a triggered alert only as emails are not allowed for Splunk app certification purposes
The macros are also used in the dashboards for this application

The vast majority of the alerts also have a macro(s) which you can customise to tweak the search results, for example the macro `splunkadmins_weekly_truncated` allows the alert, `IndexerLevel - Weekly Truncated Logs Report`, to be customised without changing the alert itself. This will make upgrading to a new version of this app more straightforward
I have attempted to provide an appropriate macro in any alert where I deemed it appropriate, feedback is welcome for any alert that you believe should have a macro or requires further improvement

## Installation
The application is designed to work on a search head or search head cluster instance, installation on the indexing tier is not required
There are a few searches that use REST API calls which are specific to the search head cluster they run on. These alerts will have to be placed on each search head or search head cluster, alternatively any server with the required search peers will also work, the relevant alerts are:
- SearchHeadLevel - Accelerated DataModels with All Time Searching Enabled
- SearchHeadLevel - Realtime Scheduled Searches are in use
- SearchHeadLevel - Realtime Search Queries in dashboards
- SearchHeadLevel - Scheduled Searches without a configured earliest and latest time
- SearchHeadLevel - Scheduled searches not specifying an index
- SearchHeadLevel - Scheduled searches not specifying an index macro version
- SearchHeadLevel - Scheduled Searches Configured with incorrect sharing
- SearchHeadLevel - Saved Searches with privileged owners and excessive write perms
- SearchHeadLevel - User - Dashboards searching all indexes
- SearchHeadLevel - User - Dashboards searching all indexes macro version
- SearchHeadLevel - Users exceeding the disk quota (recent jobs list uses a REST call so you may need to adjust the search), the SearchHeadLevel - Users exceeding the disk quota introspection is a non-search head specific alternative

The following reports also are specific to a search head or search head cluster:
- SearchHeadLevel - Alerts that have not fired an action in X days
- SearchHeadLevel - Data Model Acceleration Completion Status
- SearchHeadLevel - Macro report
- What Access Do I Have?

The following dashboards are search head or search head cluster specific:
- Data Model Rebuild Monitor
- Data Model Status

The following reports / alert must either run on the cluster master or a server where the cluster master is a peer:
- ClusterMasterLevel - Per index status
- ClusterMasterLevel - Primary bucket count per peer

## Using the application
Once the application is installed, *all* alerts are disabled by default and you can enable those you require or want to test in your local environment
If you choose not to customise the macros then many searches will search for all hosts, which will make the alerts and dashboards inaccurate!

## Which alerts should be enabled?
The alerts are all useful for detecting a variety of different scenarios which may or may not be applicable within your Splunk environment
The description field has an (extremely) simple way of determining if an alert will require action, there are three levels:
  - Low - the alert is informational and likely relates to a potential issue, these alerts may produce false alarms
  - Moderate - the alert is a warning, most likely further action will need to be taken, a moderate chance of false alarms
  - High - the alert is likely relating to something that requires action and there is a very low chance that this will create false alarms

I do not have a nice way to auto-enable various alerts excluding editing the local/savedsearches.conf or via the GUI, any contribution of a setup file would be welcome here!

## How is this application used?
In the current environment the vast majority of the alerts are enabled to detect issues, they raise automated tickets or email depending on the urgency of the specific alert.
There are a few environment characteristics that may require changes to the way the app is used, and feedback is welcome if there is a nicer way to structure the alerts/application
The overall assumption is that the admin(s) are not carefully watching the splunkd logs or the messages in the console of the monitoring server/Splunk servers

## How is this application tested?
Before 2019 the universal forwarders in use are installed on a mix of Windows, Linux & AIX servers, in 2019 and beyond the testing scope has been vastly reduced to focus primarily on Splunk enterprise servers
All heavy forwarders, and Splunk enterprise installations are Linux based, while I expect the alerts will work with only changes to the macros.conf for a Windows based environment this remains untested
The test environment for this application has a single indexer cluster and two search head clusters

## Why was this application and associated conf talk created?
Inspired by articles such as "Things I wish I knew then" and knowledge collected from various conference replays, SplunkAnswers, 200+ support tickets & nearly four years of working on a Splunk environment I decided that I would attempt to share what I have learned in an attempt to prevent others from repeating the same mistakes
There are many Splunk conf talks available on this subject in various conference replays, however my goal was to provide practical steps to implement the ideas. That is why this application exists

## Which alerts are best suited to automation?
- SearchHeadLevel - Scheduled searches not specifying an index
- SearchHeadLevel - Scheduled Searches Configured with incorrect sharing
- SearchHeadLevel - Splunk login attempts from users that do not have any LDAP roles
- SearchHeadLevel - Scheduled Searches That Cannot Run
- SearchHeadLevel - Scheduled Searches without a configured earliest and latest time
- SearchHeadLevel - Users exceeding the disk quota
- SearchHeadLevel - User - Dashboards searching all indexes
- SearchHeadLevel - Detect Excessive Search Use - Dashboard - Automated

Are all well suited to an automated email using the sendresults command or a similar function as they involve end user configuration which the individual can change/fix

## Custom search commands
Due to the current SPL not handling a particular task well, and the lookup commands not supporting regular expressions, I found that the only workable solution was to create a custom lookup command.

Two exist:
- streamfilter - based on a single (or multivalue) field name, and a single (or multivalue) field with patterns, apply the regular expression in the pattern field against the nominated field(s)
- streamfilterwildcard - identical to streamfilter except that this takes a field name with wildcards, and assumes an index-style expression, so `*` becomes `(?i)^[^_].*$`, and `example*` becomes `(?i)^example.*$`

Search help is available and these are used within the reports in this application. The Splunk python SDK version 1.6.5 is also included as this is required as part of the app, an example from the reports is:
`| streamfilterwildcard pattern=indexes fieldname=indexes srchIndexesAllowed`

Where indexes is a field name containing a list of wildcards `(_int*, _aud*)` or similar, indexes is the output field name, srchIndexesAllowed is the field name which the indexes field will be compared to.
Each entry in the pattern field will be compared to each entry in the srchIndexesAllowed field in this example

To make this command work the Splunk python SDK is bundled into the app, if the bin directory is wiped due to issues with other applications this only disables the two commands which are used in `Search Queries summary non-exact match` so far 

## KVStore Usage
Some CSV lookups are now replaced with kvstore entries due to the ability to sync the kvstore across multiple search head or search head cluster(s) via apps like [TA-SyncKVStore](https://splunkbase.splunk.com/app/3519/)

##Lookup Watcher
The Lookup Watcher is a modular input designed to work in either search head clusters or standalone Splunk instances to determine the modification time and size of all lookup files on the filesystem of the Splunk servers.
In a search head cluster the input will run on the captain only by running a rest call on each run, on a non-search head cluster it will always run.
To use this, on a non-search head cluster simply go to Settings -> Inputs and create the Lookup Watcher modular input, the name of the input does not matter, you just need to create 1 input. 
Note that the debugMode is optional and defaults to false, enabling this generates more logs for troubleshooting.

Under the more settings button choose an index to send the data to and an interval to run the script

On a search head cluster you will need to push an inputs.conf via the deployer server (if you are unsure of the syntax create one on a standalone server first)

Once done the additional logs can be used to determine how often lookups are updated and how big they are

Tested on Windows & Linux on Splunk 7.x.

Lookup Watcher generates a log file is created in $SPLUNK_HOME/var/log/splunk/ and will also be in the internal index with the name lookup_watcher.log

## Feedback?
Feel free to open an issue on github or use the contact author on the SplunkBase link and I will try to get back to you when possible, thanks!

## Release Notes
### 2.5.4
Re-release of 2.5.3 due to strange issue in SplunkBase

### 2.5.3
Lookup files are now included (zero sized), note that you will need to re-generate them after install if you overwrite the lookups used by some reports...

New macros:

`splunkadmins_audit_logs_macro_sub`

`splunkadmins_remote_macros` (this macro requires TA-webtools)

`splunkadmins_remote_roles` (this macro requires TA-webtools)

New reports:

`SearchHeadLevel - IndexesPerRole Remote Report`

`SearchHeadLevel - IndexesPerRole Report`

`SearchHeadLevel - IndexesPerRole srchIndexesallowed Report`

`SearchHeadLevel - IndexesPerRole srchIndexesdefault Report`

`SearchHeadLevel - Search Queries summary exact match 73`

`SearchHeadLevel - Search Queries summary exact match 73 by user` (uses Search Queries summary exact match 73 as base)

`SearchHeadLevel - Search Queries summary exact match 73 by index` (uses Search Queries summary exact match 73 as base)

`SearchHeadLevel - Search Queries summary non-exact match 73`

`SearchHeadLevel - IndexesPerUser Report`

Updated alerts:

`IndexerLevel - Time format has changed multiple log types in one sourcetype`

`IndexerLevel - Timestamp parsing issues combined alert`

Updated dashboard:

`issues_per_sourcetype`

Updated report:

`Updated report SearchHeadLevel - Macro report`

With new regex due to change in newer Splunk versions (credit to woodcock for the update)


Lookup file `splunkadmins_macros_temp.csv` renamed to `splunkadmins_macros.csv`


Changes for python3 compatability

Updated python SDK to 1.6.11 (from 1.6.6)

### 2.5.2
New modular input - Lookup Watcher - details in the README.md file
Introduced a new sub-menu in the navigation menu for Search Head Level "Recommended (externally hosted)" with links to external dashboards 

Updated reports:
`SearchHeadLevel - Search Queries By Type Audit Logs`
`SearchHeadLevel - Search Queries By Type Audit Logs macro version`
`SearchHeadLevel - Search Queries By Type Audit Logs macro version other`

To reduce the number of unknown queries

Updated reports:
`SearchHeadLevel - Search Queries summary exact match`
`SearchHeadLevel - Search Queries summary non-exact match`

To improve the statistics around indexes found

### 2.5.1
Updated alert - `SearchHeadLevel - Scheduled Searches That Cannot Run` tweak to find more results

Updated dashboard `issues per sourcetype` to handle message becoming event_message in newer Splunk versions (7.1 or 7.2)

Updated macros `splunkadmins_shutdown_list`, `splunkadmins_shutdown_keyword`, `splunkadmins_shutdown_time`, `splunkadmins_transfer_captain_times` to handle message becoming event_message in newer Splunk versions (7.1 or 7.2)

Updated python files streamfilter/streamfilterwildcard to import lib relative to the current app name

Updated alerts / reports:
 - `AllSplunkLevel execprocessor errors`
 - `AllSplunkLevel - TCP Output Processor has paused the data flow`
 - `AllSplunkEnterpriseLevel - Detect LDAP groups that no longer exist`
 - `AllSplunkEnterpriseLevel - Email Sending Failures`
 - `AllSplunkEnterpriseLevel - File integrity check failure`
 - `AllSplunkEnterpriseLevel - Non-existent roles are assigned to users`
 - `AllSplunkEnterpriseLevel - Replication Failures`
 - `AllSplunkEnterpriseLevel - TCP or SSL Config Issue`
 - `AllSplunkEnterpriseLevel - Unable to dispatch searches due to disk space`
 - `DeploymentServer - btool validation failures occurring on deployment server`
 - `DeploymentServer - Unsupported attribute within DS config`
 - `ForwarderLevel - crcSalt or initCrcLength change may be required`
 - `ForwarderLevel - Splunk Universal Forwarders Exceeding the File Descriptor Cache`
 - `IndexerLevel - Buckets are been frozen due to index sizing`
 - `IndexerLevel - IndexConfig Warnings from Splunk indexers`
 - `IndexerLevel - Index not defined`
 - `IndexerLevel - Peer will not return results due to outdated generation`
 - `IndexerLevel - Time format has changed multiple log types in one sourcetype`
 - `IndexerLevel - Too many events with the same timestamp`
 - `IndexerLevel - Valid Timestamp Invalid Parsed Time`
 - `SearchHeadLevel - KVStore Or Conf Replication Issues Are Occurring`
 - `SearchHeadLevel - LDAP users have been disabled or left the company cleanup required`
 - `SearchHeadLevel - Scheduled Searches That Cannot Run`
 - `SearchHeadLevel - Scheduled searches failing in cluster with 404 error`
To handle message becoming event_message in newer Splunk versions (7.1 or 7.2)

### 2.5.0
New dashboard `HEC Performance` (original from [camrunr's github](https://github.com/camrunr/hec_perf_report/blob/master/hec_perf_report.xml))

New macro - `splunkadmins_shutdown_keyword`

New report - `IndexerLevel - Knowledge bundle upload stats`

Updated alert - `AllSplunkEnterpriseLevel - Replication Failures` with new criteria and excluded shutdowns

Updated alert - `AllSplunkEnterpriseLevel - Splunk Scheduler skipped searches and the reason` to handle another skipped scenario

Updated alert - `AllSplunkEnterpriseLevel - Splunk Servers with resource starvation` with new comments 

Updated alert - `SearchHeadLevel - Detect MongoDB errors` with update to handle tstats issue in Splunk (issue #3 in github)

Moved splunklib into "lib" directory of app as per updated appinspect recommendations

### 2.4.9
Updated alert - `SearchHeadLevel - Detect MongoDB errors` to include " W " based on git feedback

### 2.4.8
New alert - `ForwarderLevel - Splunk HEC issues`

New dashboard - `Lookups in use finder`

New macro - `splunkadmins_license_usage_source`

New report - `IndexerLevel - Maximum memory utilisation per search`

New report - `SearchHeadLevel - Lookup updates within SHC`

New report - `SearchHeadLevel - Maximum memory utilisation per search`

New report - `SearchHeadLevel - Detect Excessive Search Use - Dashboard - Automated`

Updated alert - `AllSplunkEnterpriseLevel - Replication Failures` to match more results

Updated alert - `ForwarderLevel - Splunk HTTP Listener Overwhelmed` comment/description update

Updated dashboard - `Rolled buckets by index` - to no longer hardcode Linux paths to the license usage log

Updated dashboard - `Heavy Forwarders Max Data Queue Sizes by name` to use the thruput in the metrics.log

### 2.4.7
New README (README.md replaces README)
New dashboard `Detect excessive search usage`
New dashboard `Cluster Master Jobs`
New dashboard `Knowledge Objects by app` (and drilldown dashboard)
New report - `IndexerLevel - Corrupt buckets via DBInspect`
New report - `SearchHeadLevel - Detect changes to knowledge objects`
New report - `SearchHeadLevel - Detect changes to knowledge objects directory`
New report - `SearchHeadLevel - Detect changes to knowledge objects non-directory`
Updated alert `ForwarderLevel - Splunk Universal Forwarders Exceeding the File Descriptor Cache` (comment update)
Updated alert `IndexerLevel - Uneven Indexed Data Across The Indexers` to handle a varying number of indexers
Updated various reports to include the `splunkadmins_restmacro`, this ensures `splunk_server=local` is used where appropriate
Updated dashboard `heavyforwarders_max_data_queue_sizes_by_name`, now has a filter for hosts to look at, corrected TCPOut KB per second panel
Updated macro `splunkadmins_splunkd_source` now defaults to `*splunkd.log` (previously `/opt/splunk/var/log/splunk/splunkd.log`)
Updated macro `splunkadmins_mongo_source` now defaults to `*mongod.log` (previously `/opt/splunk/var/log/splunk/mongod.log`)
Updated report `SearchHeadLevel - Search Queries summary exact` to remove the mvexpand and selfjoin (replaced by stats)

### 2.4.6
New alert - AllSplunkLevel - Data Loss on shutdown
New macro - whataccessdoihave - can be used with | `whataccessdoihave` by users
New report - SearchHeadLevel - Dashboard load times
New report - SearchHeadLevel - Scheduled searches status
Updated dashboard - Troubleshooting Resource Usage Per User Drilldown - now uses `search_et/search_lt`
Removed report - What access do I have? (Replaced by macro/What access do I have without REST)

Upgraded Splunk python SDK to 1.6.6, note if this causes problems with other applications removing the bin directory only disables the "Search Queries summary non-exact match" report

### 2.4.5
Minor corrections

Updated SearchHeadLevel - Search Queries By Type Audit Logs - minor tweak to macroWithIndexClause
Updated SearchHeadLevel - Search Queries By Type Audit Logs macro version - minor tweak to macroWithIndexClause and hasMacro
Updated SearchHeadLevel - Search Queries By Type Audit Logs macro version other - minor tweak to macroWithIndexClause and hasMacro

### 2.4.4
New command - streamfilter
New command - streamfilterwildcard
New dashboard - Troubleshooting Resource Usage Per User
New dashboard - Troubleshooting Resource Usage Per User Drilldown
New report - SearchHeadLevel - Index access list by user
New report - SearchHeadLevel - Index list report
New report - SearchHeadLevel - Role access list by user
New report - SearchHeadLevel - Scheduled Search Efficiency
New report - SearchHeadLevel - Search Queries Per Day Audit Logs
New report - SearchHeadLevel - Search Queries By Type Audit Logs
New report - SearchHeadLevel - Search Queries By Type Audit Logs macro version
New report - SearchHeadLevel - Search Queries By Type Audit Logs macro version other
New report - SearchHeadLevel - Search Queries summary exact match
New report - SearchHeadLevel - Search Queries summary non-exact match
New report - SearchHeadLevel - Users with auto-finalized searches
New report - What Access Do I Have Without REST? To work without the `dispatch_rest_to_indexers`
Updated alert - AllSplunkEnterpriseLevel - Email Sending Failures - to make it easier to automate
Updated alert - SearchHeadLevel - Captain Switchover Occurring - to ignore a harmless warning message (NOT_LEADER)
Updated alert SearchHeadLevel - Scheduled Searches That Cannot Run to no longer ignore map alerts from this app
Updated alert - SearchHeadLevel - Scheduled searches not specifying an index macro version - to use an improved macro match
Updated alert - SearchHeadLevel - User - Dashboards searching all indexes macro version - to use an improved macro match
Updated dashboard Troubleshooting indexer CPU as sort was not working
Updated report - SearchHeadLevel - Macro report - to use `splunk_server=local`
Updated report - What Access Do I Have? to use `splunk_server=local`
Updated report - What Access Do I Have Without REST? to supply index list

### 2.4.3
A very minor release, the app inspect CLI and REST API provided different results on what needed to be fixed

Updated alert SearchHeadLevel - Users exceeding the disk quota introspection to not throw an error in the map command for no results
Updated alert SearchHeadLevel - Users exceeding the disk quota to not throw an error in the map command for no results

### 2.4.2
A very minor release, the app inspect badge does not allow external dependencies so changing 1 alert to get the badge

Updated alert SearchHeadLevel - Users exceeding the disk quota introspection to comment out sendresults command

### 2.4.1
Introduced an updated navigation menu to navigate around the alerts, reports and dashboards available in the app
Changed label of all dashboards to have Dashboard - ... this is just to make the navigation menu work as expected

New alert IndexerLevel - Buckets changes per day
New alert IndexerLevel - Timestamp parsing issues combined alert
New report SearchHeadLevel - Audit log search example only
Updated alert IndexerLevel - Future Dated Events that appeared in the last week to +10y instead of +20y
Updated alert IndexerLevel - Indexer Queues May Have Issues - to work with multiple pipelines
Updated alert IndexerLevel - Buckets rolling more frequently than expected with an improved regex
Updated alert SearchHeadLevel - Captain Switchover Occurring - to ignore manual captain transfers
Corrected alert SearchHeadLevel - Determine query scan density with a relevant query

Note 2.4.0 was never released

### 2.3.9
Updated alert SearchHeadLevel - Detect searches hitting corrupt buckets to detect 1 more variation of the issue
Updated alert SearchHeadLevel - Users exceeding the disk quota to include username
Updated alert SearchHeadLevel - Scheduled Searches That Cannot Run to ignore the new report (SearchHeadLevel - Users exceeding the disk quota introspection)
Updated report ForwarderLevel - Forwarders connecting to a single endpoint for extended periods (and UF level version) to use the hostname/name parameters
Renamed alert IndexerLevel - ERROR from linebreaker to IndexerLevel - Data parsing error
New report SearchHeadLevel - Users exceeding the disk quota introspection
New report SearchHeadLevel - Users exceeding the disk quota introspection cleanup

### 2.3.8
New reports for diagnosing forwarder issues, alerts around bucket corruption and peer connection failures
New dashboards for troubleshooting sourcetypes or buckets rolled per day
Updated all alerts with an investigationQuery to use `index=*` explicitly rather than assume the admin has all indexes listed in the indexes searched by default list

Update summary:
New alert - IndexerLevel - Detect bucket corruption
New alert - SearchHeadLevel - Indexer Peer Connection Failures
Updated alert ClusterMasterLevel - Per index status to 5 minute intervals for certification purposes
Renamed alert IndexerLevel - Detect bucket corruption to a report IndexerLevel - Report on bucket corruption (refer to IndexerLevel - Unclean Shutdown - Fsck for an alert)
New report - ForwarderLevel - Forwarders connecting to a single endpoint for extended periods
New report - ForwarderLevel - Forwarders connecting to a single endpoint for extended periods UF level
New report - SearchHeadLevel - Determine query scan density
New report - SearchHeadLevel - Detect searches hitting corrupt buckets
New dashboard - Issues per sourcetype, a combination of timestamp parsing, future based and past data searches to look at a single problematic sourcetype
New dashboard - Rolled buckets by index, a dashboard to assist with determing which index is rolling the most buckets

### 2.3.5
Update summary:
Updated IndexerLevel - Cold data location approaching size limits to handle only maxTotalDataSizeMB been set
Updated Future Dated Events that appeared in the last week to use +10y and 7.1 rejects +20y
Corrected AllSplunkEnterpriseLevel - TCP or SSL Config Issue to remove extra ( symbol
Corrected SearchHeadLevel - User - Dashboards searching all indexes macro version to refer to correct lookup name
Corrected dashboard for troubleshooting indexer CPU to handle standalone server
Inclusion of alternative app icons to work in 7.1

### 2.3.4
Update summary:
Updated SearchHeadLevel - Scheduled searches not specifying an index to exclude 1 additional type of search
Updated SearchHeadLevel - KVStore Or Conf Replication Issues Are Occurring to detect a disconnected member scenario
Updated Troubleshooting indexer CPU & drilldown dashboards to include commmas and the search head field (to make it easier to update to search head instead of indexer hosts)

### 2.3.3
Update summary:
New alert SearchHeadLevel - Disabled modular inputs are running
Updated SearchHeadLevel - Detect MongoDB errors to timechart to have no limit on the number of hosts involved
Updated the shutdown macros to find one additional scenarios

### 2.3.2
Due to resourcing issues on the search heads this includes a few warnings/errors related to performance issues

Update summary:
New alert AllSplunkEnterpriseLevel - Splunk Servers with resource starvation
New alert IndexerLevel - S2SFileReceiver Error
New alert SearchHeadLevel - Captain Switchover Occurring
Updated ForwarderLevel - Splunk Universal Forwarders that are time shifting to include "System time went backwards by..."
Updated IndexerLevel - Failures To Parse Timestamp Correctly (excluding breaking issues) to show when the failure related to been outside the acceptable time window
Updated SearchHeadLevel - User - Dashboards searching all indexes to simplify regex (ignore anything starting with a pipe symbol)
Updated SearchHeadLevel - User - Dashboards searching all indexes macro version to simplify regex (ignore anything starting with a pipe symbol)
Corrected AllSplunkEnterpriseLevel - sendmodalert errors to not show random `savedsearch_names` when no match is found
Corrected SearchHeadLevel - Alerts that have not fired an action in X days to only show alerts relevant to the current search head/cluster

### 2.3.1
Update summary:
New alert AllSplunkEnterpriseLevel - Non-existent roles are assigned to users
New alert IndexerLevel - Index not defined
New alert IndexerLevel - Search Failures
New alert SearchHeadLevel - Scheduled searches not specifying an index macro version (detect lack of index= with 1 level of macro expansion)
New alert SearchHeadLevel - Saved Searches with privileged owners and excessive write perms (detect 1 way of accessing data outside your level of access)
New alert SearchHeadLevel - User - Dashboards searching all indexes macro version
New report SearchHeadLevel - Macro report (required by "macro version" alerts)
Updated AllSplunkEnterpriseLevel - TCP or SSL Config Issue to include an additional scenario as reported by a customer
Updated SearchHeadLevel - Scheduled searches not specifying an index to not find searches with macros and to include example query
Updated SearchHeadLevel - Scheduled Searches That Cannot Run to make the message field accurate in all situations
Updated SearchHeadLevel - User - Dashboards searching all indexes to include example query to find indexes, and to not find macro-based queries
Corrected AllSplunkLevel - Unable To Distribute to Peer
Corrected IndexerLevel - Failures To Parse Timestamp Correctly (excluding breaking issues) to correctly exclude broken events & to handle newer 7.0.2 errors

### 2.3.0
Minor updates to a few alerts and a new alert

Update summary:
New alert AllSplunkEnterpriseLevel - Detect LDAP groups that no longer exist
New alert ClusterMasterLevel - Per index status
New report ClusterMasterLevel - Primary bucket count per peer
Updated AllSplunkEnterpriseLevel - TCP or SSL Config Issue to find most recent (not oldest example)
Updated AllSplunkEnterpriseLevel - Splunk Scheduler skipped searches and the reason to exclude the timewindow upto 10 minutes post-shutdown of an indexer
Updated AllSplunkLevel - TCP Output Processor has paused the data flow to use a stats command instead of raw/host information
Updated DeploymentServer - Unsupported attribute within DS config - to find most recent (not oldest example)
Updated IndexerLevel - Failures To Parse Timestamp Correctly (excluding breaking issues) - to find most recent (not oldest example)
Updated SearchHeadLevel - Detect MongoDB errors - mild tweak to output data, added customisation macros
Updated SearchHeadLevel - Scheduled Searches That Cannot Run - to detect errors in splunkd related to saved searches
Corrected SearchHeadLevel - User - Dashboards searching all indexes - a newline resulted in it working in search but not via the scheduler!

### 2.2
Not released, combined with 2.3.0
Attempt to reduce false alarms and improve investigationQuery searches
Created macros for shutdown events for indexers/search heads/enterprise servers for excluding false alarms related to restarts

Update summary:
New macro `splunkadmins_shutdown_list`
New macro `splunkadmins_shutdown_time`
Updated AllSplunkEnterpriseLevel - Splunk Scheduler skipped searches and the reason - to use shutdown macro
Updated AllSplunkEnterpriseLevel - Splunk Scheduler excessive delays in executing search - to use shutdown macro
Updated AllSplunkLevel - TCP Output Processor has paused the data flow - to use shutdown macro
Updated AllSplunkLevel - Unable To Distribute to Peer - to use shutdown macro
Updated ForwarderLevel - Splunk forwarders are having issues with sending data to indexers - to use shutdown macro
Updated ForwarderLevel - SplunkStream Errors - to use shutdown macro
Updated ForwarderLevel - Unusual number of duplication alerts - to use shutdown macro & changed the alert to fire on >10 results per host
Updated IndexerLevel - Weekly Truncated Logs Report - hostnames wildcarded to deal with short names (for syslog for example)
Updated SearchHeadLevel - Detect MongoDB errors - timespan increased to 10 minutes and 5 minutes produces false alarms, and added shutdown macro
Updated SearchHeadLevel - KVStore Or Conf Replication Issues Are Occurring - to use shutdown macro

### 2.1
Added macros which can be customised to the majority of alerts, this reduces the need to customise the alert itself and should make upgrading to new versions of the application easier...

Update summary:
New alert AllSplunkEnterpriseLevel - Unable to dispatch searches due to disk space
New alert IndexerLevel - Unclean Shutdown - Fsck
New macros - various macros introduced due to customer feedback about the requirement to customise the alerts
Updated SearchHeadLevel - Users exceeding the disk quota to list top 10 consumers of disk
Updated SearchHeadLevel - Scheduled Searches That Cannot Run (to ignore the above)
Updated IndexerLevel - Failures To Parse Timestamp Correctly (excluding breaking issues) to list sources per sourcetype/host
Updated ForwarderLevel - Splunk Insufficient Permissions to Read Files to include new macro, hint, invesQuery & to improve the accuracy
Updated SearchHeadLevel - Detect MongoDB errors (customer feedback, includes F/fatal errors now)
README and description updates for searches

### 2.0
Multiple searches now have an "investigationQuery" in them, the idea is that you can copy and paste the output into a search window and see results relevant to the particular alert
The last few releases have been attempting to reduce false alarms from alerts related to server restarts

Update summary:
New alert IndexerLevel - Cold data location approaching size limits
New/renamed alert AllSplunkEnterpriseLevel - Splunk Scheduler excessive delays in executing search
New/renamed alert AllSplunkEnterpriseLevel - Splunk Scheduler skipped searches and the reason
Updated AllSplunkLevel - Time skew on Splunk Servers
Updated IndexerLevel - Future Dated Events that appeared in the last week
Updated IndexerLevel - Time format has changed multiple log types in one sourcetype
Updated IndexerLevel - Failures To Parse Timestamp Correctly (excluding breaking issues)
Updated IndexerLevel - Weekly Broken Events Report
Updated IndexerLevel - Weekly Truncated Logs Report
Updated IndexerLevel - Old data appearing in Splunk indexes
Updated IndexerLevel - Valid Timestamp Invalid Parsed Time
Updated IndexerLevel - Too many events with the same timestamp
Updated IndexerLevel - Large multiline events using `SHOULD_LINEMERGE` setting
Updated SearchHeadLevel - Users exceeding the disk quota
Updated IndexerLevel - Indexer Queues May Have Issues (to be less sensitive to indexqueue issues)
Corrected AllSplunkLevel - TCP Output Processor has paused the data flow
Corrected ForwarderLevel - Splunk Universal Forwarders that are time shifting
Removed AllSplunkEnterpriseLevel - Splunk Servers with time skew (replaced by Time skew on Splunk Servers)
Removed SearchHead Level - Splunk Scheduler excessive delays in executing search (renamed)
Removed SearchHeadLevel - Splunk Scheduler Skipped Searches and the reason (renamed)

### 1.9
New macro `splunkadmins_mongo_source`
New alert IndexerLevel - Too many events with the same timestamp
New alert SearchHeadLevel - Detect MongoDB errors
New dashboard Data Model Status
New dashboard Data Model Rebuild Monitor
Updated AllSplunkEnterpriseLevel - Email Sending Failures to include to the toaddress
Updated SearchHeadLevel - Splunk Users Violating the Search Quota to detect an alternative log message
Updated Scheduled searches not specifying an index
Updated SearchHeadLevel - Scheduled Searches without a configured earliest and latest time
Updated ForwarderLevel - File Too Small to checkCRC occurring multiple times, to handle spaces in filename
Updated ForwarderLevel - crcSalt or initCrcLength change may be required
Updated IndexerLevel - Indexer Queues May Have Issues to be less sensitive
Updated IndexerLevel - Splunk Indexers Losing Contact With Master for an additional scenario
Updated AllSplunkEnterpriseLevel - ulimit on Splunk enterprise servers is below 8192 to improve emails
Corrected AllSplunkLevel - Splunk forwarders that are not talking to the deployment server

### 1.8
New alert IndexerLevel - Peer will not return results due to outdated generation
New alert SearchHeadLevel - Scheduled searches failing in cluster with 404 error
Updated ForwarderLevel - File Too Small to checkCRC occurring multiple times to have the correct dispatch application
Updated AllSplunkEnterpriseLevel - Email Sending Failures with saved search name
Updated IndexerLevel - Indexer replication queue issues to some peers to be less sensitive as this cannot be tuned in 7.0.0
Updated AllSplunkLevel - Unable To Distribute to Peer to include the peer name
Updated IndexerLevel - Indexer Queues May Have Issues to ensure it fires when neccesary but is not too noisy (this may require tuning)
Corrected ForwarderLevel - Bandwidth Throttling Occurring, this alert was not working as expected

### 1.7
New macro `splunkadmins_splunkuf_source`
New alert "AllSplunkEnterpriseLevel - TCP or SSL Config Issue" for detecting when the listener ports fail to start on a HF/Indexer
Updated macro splunkindexerhostsvalue to include `splunk_server=`
Updated searches to use (`splunkadmins_splunkd_source`) in brackets so it looks valid when expanded (and allows a future OR/NOT statement to be added before or after with no unexpected side effects)
Updated a few comments and improved some searches to narrow down the required hosts/sources/sourcetypes
Removed unused macro splunkenterprisehostsvalue
Removed hardcoded references to the location of splunkd.log file and replaced with `splunkadmins_splunkd_source` macro
Removed a few unnessary fields/fixed some other minor issues within the file

### 1.6
Removed "Splunk Alert failures" and updated "AllSplunkEnterpriseLevel - sendmodalert errors", also updated "Time format has changed" alert to have more clear output via email

### 1.5
Updated Splunk Alert Failures alert and the Time format has changed alerts to have more clear output via email
Simplified "Scheduled Searches without a configured earliest and latest time", and "Scheduled searches not specifying an index"

### 1.4
Two new alerts LicenseMaster - Duplicated License Situation, DeploymentServer - Unsupported attribute within DS config
Simplified Scheduled Searches without a configured earliest and latest time, and Scheduled searches not specifying an index
Created a macro `splunkadmins_splunkd_source` for Windows users or others using non-standard Splunk installation directories

### 1.0 to 1.3
Creation of app, addition of icons and removal of email functionality from the app for Splunk certification purposes

## Other
Icons made by [Freepik](http://www.freepik.com) from www.flaticon.com is licensed by [Creative Commons BY 3.0](http://creativecommons.org/licenses/by/3.0)

### Misc testing notes for SearchHeadLevel - Detect changes to knowledge objects
calcfields:
/data/props/calcfields
/servicesNS/admin/search/data/props/calcfields (GUI goes via /manager/ first)
/services/data/props/calcfield
https://localhost:8089/servicesNS/nobody/search/configs/conf-props?count=0 <-- did not work/no data found

Saved searches:
/servicesNS/nobody/search/configs/conf-savedsearche
/services/configs/conf-savedsearches
/services/saved/searches
/services/admin/savedsearch
/en-US/splunkd/__raw/servicesNS/admin/search/saved/searches/

dashboards:
/services/data/ui/views
/en-US/splunkd/__raw/servicesNS/admin/search/data/ui/views
/services/admin/views/

fieldaliases:
/servicesNS/admin/search/data/props/fieldaliases (GUI goes via /manager/ first)
/servicesNS/admin/search/data/props//fieldaliases
/services/admin//fieldaliases
/services/configs/conf-props

field extractions:
/servicesNS/admin/search/data/props/extractions
/services/admin/props-extract
/services/configs/conf-props

fieldtransforms:
/servicesNS/admin/search/data/transforms/extractions
/services/admin//transforms-extract
/services/configs/conf-transforms

workflowactions:
/data/ui/workflow-actions
/services/admin//workflow-actions/TestWorkflow
/services/configs/conf-workflow_actions

sourcetype renaming:
/data/props/sourcetype-rename
/admin//sourcetype-rename
/services/configs/conf-props

tags:
/configs/conf-tags
/admin/tags
/saved/ntags
/saved/fvtags

eventtypes:
/saved/eventtypes
/admin//eventtypes
/configs/conf-eventtypes

navMenu:
/data/ui/nav
/admin/nav/

datamodel:
/datamodel/model
/configs/conf-datamodels
/admin//datamodeledit
/admin//datamodel-files

kvstore:
/storage/collections/config
/configs/conf-collections
/admin//collections-conf

/configs/conf-viewstates
Skipped

times:
/data/ui/times
/configs/conf-times
/admin//conf-times

UI panels:
/data/ui/panels
/configs/conf-panels

automatic lookups:
/data/transforms/lookups
/admin//transforms-lookup
/services/configs/conf-transforms

lookup definitions:
/data/props/lookups
/admin//props-lookup
/services/configs/conf-props

macros:
/configs/conf-macros
/data/macros
/admin/macros
