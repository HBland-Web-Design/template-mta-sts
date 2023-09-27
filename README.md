# Template MTA-STS Hosted
Template Repository for Hosting `mta-sts.example.com/.well-known/mta-sts.txt` with GitHub Pages

For more information around MTA-STS and the IETF RFC 8461 Standard see linked Standard [Here](https://www.rfc-editor.org/info/rfc8461)

## How To

- Start by forking this Repository
- Edit the .well-known/mta-sts.txt file to include your mx directives as set in your DNS 
- Set the mode you would like to implement the policy in (See [Mode Explained for details](#mode-explained))
- Enable GitHub Pages and set the Root of your Main Branch as the root directory for your site
- Create DNS records Described in [DNS Additions](#dns-additions)
- Enable Enforce HTTPS for GitHub Pages

### DNS Additions
- Create a CNAME Record `mta-sts.example.com` pointing to your GitHub Pages site
```
Hostname: mta-sts.example.com
TTL: Default
Type: CNAME
Value: example.github.io
```

- Create a TXT Record `_mta-sts.example.com` like below
```
Hostname: _mta-sts.example.com
TTL: 3600
Type: TXT
Value: v=STSv1; id={Choose a String Value for each policy Update}Z;
```

### MODE Explained
`enforce`
Sending MTAs MUST NOT deliver messages to hosts that fail MX matching or certificate validation or do not support STARTTLS.

`testing`
Sending MTAs that implement the TLSRPT (TLS Reporting) specification [RFC8460](https://www.rfc-editor.org/info/rfc8460) send a report indicating policy failures (so long as TLSRPT is implemented by the recipient and sender domains) 
However messages may be delivered as if MTA-STS validation failure had not happened.

`none`
Sending MTAs should treat the Policy Domain as though it does not have any active policy


### Removing MTA-STS

1. Publish a new policy with `mode` equal to `none` and a small `max_age`
```
version: STSv1
mode: none
max_age: 3600
```
2. Publish a new TXT record to trigger fetching of the new policy
```
Hostname: _mta-sts.example.com
TTL: 3600
Type: TXT
Value: v=STSv1; id={Choose a String Value for each policy Update}Z;
```
3. When all previously served policies have expired safely remove the `TXT` record and `HTTPS` endpoint.
    - Normally this is the time the previously published policy was last served plus that policy’s `max_age`,
    - Note that policies older than the previously published policy may have been served with a greater `max_age` than the previously published policy, allowing for overlapping policy caches