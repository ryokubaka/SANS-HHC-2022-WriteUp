# Suspicious PCAP

### Question 1. There are objects in the PCAP file that can be exported by Wireshark and/or Tshark. What type of objects can be exported from this PCAP?`

In Wireshark, we go to `File -> Export objects -> HTTP` and see the following screen:

![](1-export.png)

Answer: **http**

### Question 2. What is the file name of the larget file we can export?

We can refer to the screenshot above.

Answer: **app.php**

### Question 3. What packet number starts that app.php file?

We can refer to the screenshot above.

Answer: **687**

### Question 4. What is the IP of the Apache server?

In the main Wireshark screen, go to packet 687. We can see that this is a HTTP OK response, which means it's the server responding to a client. This means the Apache server is the source IP address.

![](2-apacheip.png)

Answer: **192.185.57.242**

### Question 5. What file is saved to the infected host?

Poking around the rest of the pcap, we don't find anything indicated a download. Since we know the host visited app.php, we download it from the "Export Objects" page and inspect the code.  At the very bottom of the code, we see the following:

```php
    let blob1 = new Blob([byteArray], {type: 'application/octet-stream'});

    saveAs(blob1, 'Ref_Sept24-2020.zip');
```

Answer: **Ref_Sept24-2020.zip**

### Question 6. Attackers used bad TLS certificates in this traffic. Which countries were they registered to? Submit the names of the countries in alphabetical order separated by a comma (Ex. Norway, South Korea).

1. To get this, I used the `tls` filter and looked for a **Server Hello** packet.
2. Then, from the bottom section, I filtered down to where it identifies a certificate Country.  Right click on that, then select "Prepare as a filter"

![](3-countrypreparefilter.png)

3. We find that the filter is **x509sat.CountryName**. I right click on the table headers and select "Column Preferences..."

4. I create a new column, named "TLS Country", set the Type to "Custom", and put in the filter I found.

![](4-countrycolumn.png)

5. Now I sort on the new column and see the follow country codes used: `US`,`SS`,`IE`,`IL`. Using simple process of elimination, we know it's Israel (IL) and South Sudan (SS).

Answer: **Israel, South Sudan**

### Question 7. Is the host infected (Yes/No)?

Well, they already hinted in question 5 that the host IS infected.

Answer: **Yes**

We're done!