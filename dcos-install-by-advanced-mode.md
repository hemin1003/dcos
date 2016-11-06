IP Detect

echo $\(ip addr \| grep "inet &lt;IP网段，如：192.168.83&gt;" \| awk '{print $2}' \| awk -F '\/' '{print $1}'\)



