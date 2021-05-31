# crowdstrike-spotlight-slack-nagger
Nags users about outstanding application vulnerabilities found by Crowdstrike Spotlight so they patch their software.

![slack example](.github/readme/screenshot.png)

## Instructions

1. Download a release of csn
2. Create a configuration file

```yaml
slack:
  token: "XXX"

falcon:
  clientid: "XXX"
  secret: "XXX"
  cloud_region: "eu-1"

email_domain: "mycompany"

message: |
  *:warning:  We found security vulnerabilities on your device(s)*
  Hi {{ .Slack.Profile.FirstName }} {{ .Slack.Profile.LastName }}! One or more of your devices seem to be vulnerable.
  Luckily we noticed there are patches available. :tada:
  Can you please update following software as soon as possible?

  {{ range $device := .User.Devices }}
  :computer: {{ $device.MachineName }}
  {{ range $vuln := $device.Findings }}
    `{{ $vuln.ProductName }}`
  {{ end }}
  {{ end }}

  Please update them as soon as possible. In case of any issues, hop into *#security*.
  Thank you! :wave:
```
3. Run `csn -config=your-config.yml`.
4. See it popup in Slack!