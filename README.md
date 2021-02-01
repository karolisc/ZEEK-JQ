# ZEEK-JQ
We Start off by converting a PCAP with 802.11 wireless packets to JSON
<pre><code>sudo tshark -T ek -r "pcap_file"<pre><code>
Useful Zeek jq commands

# print out source host and port, destination host and port
<pre><code>cat conn.log | jq -j '.["id.orig_h"], ", ", .["id.orig_p"], ", ",.["id.resp_h"], ", ", .["id.resp_p"], "\n"'

# print out source host and port, destination host and port along with connection state.  Look for port 22, sort and unique
<pre><code>zcat conn.* | jq -j '.["id.orig_h"], ", ", .["id.orig_p"], ", ",.["id.resp_h"], ", ", .["id.resp_p"], ", ", .["conn_state"], "\n"' | grep -P " 22, " | sort -u

# print smb_share information
<pre><code>cat smb_mapping.log | jq -j '.["id.orig_h"], "\t", .["id.resp_h"], "\t", .["path"], "\t", .["share_type"],"\n"' | grep -v null | sort -u

# print syslog.log messages (source host and message) if the facility field is equal to "LOCAL0"
<pre><code>zcat syslog* | jq -j 'select(.facility == "LOCAL0") | .["id.orig_h"], "\t", .["message"], "\n"'

# print the originating host of a DNS query for the query type "TKEY"
<pre><code>zcat dns* | jq -j 'select(.qtype_name == "TKEY") | "Originator", ": ", .["id.orig_h"], "\n"'

# Print a column header with source, destination and duration names.  Select the source and destination from conn.log where the duration is # greater than 10000 seconds.
<pre><code>echo -e "source\t\tdestination\t\tduration" && zcat conn.* | jq -j 'select(.duration > 10000) | .["id.orig_h"], "\t", .["id.resp_h"], "\t", .duration, "\n"'

# Print and sort all users who's accounts were locked out from Zeek kerberos log archive.
<pre><code>zcat kerberos.* | jq -j 'select(.error_msg == "KDC_ERR_CLIENT_REVOKED") | .["id.orig_h"], "\t", .["error_msg"], "\t", .["client"], "\n"' | grep -v null | sort | uniq -c | sort -n -r

# Print log messages associated with a specific zeek uid from all archived logs
<pre><code>ls -la *.gz | awk '{print $9}' | while read line; do zcat $line |jq 'select(.uid == "Cb0Tr91RJWdZQCia4")' ;done 2>/dev/null
