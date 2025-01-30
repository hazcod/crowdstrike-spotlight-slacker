# 🤖 security-slacker
Pokes users on Slack about outstanding risks found by Crowdstrike Spotlight or vmware Workspace ONE so they can secure their own endpoint.

Self-service security culture! :partying_face:

Slack message for the user:

![slack example](.github/readme/user.png)

Slack overview message for the security user:

![slack example](.github/readme/overview.png)

## Heroku

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

## Instructions

1. Tag your Falcon hosts with `email/user/company/com` if their email is `user@company.com`.
2. Assign compliance policies to your devices in Workspace ONE. 
3. Fetch a binary release or Docker image from [Releases](https://github.com/hazcod/security-slacker/releases).
4. Create a Falcon API token to use in `API Clients and Keys` with `Read` permission to `Hosts` and `Spotlight`.
5. Create Workspace ONE oauth2 API credentials with a read-only role.
6. Create a configuration file:
```yaml
slack:
  # slack bot token
  token: "XXX"
  # Slack user that receives  messages if the user is not found
  security_user: ["security@mycompany.com"]
  # skip sending a security overview if there is nothing to mention
  skip_no_report: true
  # don't send a message to the user if 'Vacationing' status is set
  skip_on_holiday: true

# falcon crowdstrike
falcon:
  # falcon api credentials
  clientid: "XXX"
  secret: "XXX"
  # your falcon SaaS cloud region
  cloud_region: "eu-1"
  # skip vulnerabilities without available patches
  skip_no_mitigation: true
  # what severity classes you want to skip
  skip_severities: ["low"]
  # minimum CVE base score to report
  min_cve_base_score: 0
  # the CVEs you want to ignore
  skip_cves: ["CVE-2019-15315"]
  # the minimum exprtAI severity you want to filter for
  min_exprtai_severity: medium

# vmware workspace one
ws1:
  # the api endpoint of your Workspace ONE instance, eg. "https://asXXXX.awmdm.com/api/"
  api_url: "https://xxx.awmdm.com/api/"
  # your Workspace ONE oauth2 credentials
  # Groups & Settings > Configurations > Search for "oauth" > Click > Add with a Reader role
  client_id: "XXX"
  client_secret: "XXX"
  # the location of your Workspace ONE tenant, see 'Region-specific Token URLs'
  # https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/services/UEM_ConsoleBasics/GUID-BF20C949-5065-4DCF-889D-1E0151016B5A.html
  auth_location: "emea"
  # what policies you want to skip
  # leave user or policy blank to ignore it
  skip:
  - policy: "Disk Encryption"
    user: "some_special_user@company.com"

# email domains used in your Slack workspace for filtering
# e.g. for a Slack account user@mycompany.com
email:
  domains: ["mycompany.com"]
  # any users that shouldn't be in MDM or EDR
  whitelist:
  - foo@company.com

# what is sent to the user in Go templating
templates:
  user_message: |
    *:warning:  We detected security issues on your device(s)*
    Hi {{ .Slack.Profile.FirstName }} {{ .Slack.Profile.LastName }}!

    {{ if not (eq (len .Falcon.Devices) 0) }}
    One or more of your devices seem to be vulnerable.
    Luckily we noticed there are patches available. Please install following patches:
    {{ range $device := .Falcon.Devices }}
    :computer: {{ $device.MachineName }}
    {{ range $vuln := $device.Findings }}
      `{{ $vuln.ProductName }}`
    {{ end }}
    {{ end }}
    {{ end }}

    {{ if not (eq (len .WS1.Devices) 0) }}
    One or more of your devices seem to be misconfigured in an insecure way.
    Please check the below policies which are violated:
    {{ range $device := .WS1.Devices }}
    :computer: {{ $device.MachineName }}
    {{ range $finding := $device.Findings }}
    - :warning: {{ $finding.ComplianceName }}
    {{ end }}
    {{ end }}
    {{ end }}

    Please resolve those issues as soon as possible. In case of any issues, hop into *#security*.
    Thank you! :wave:

  security_overview_message: |

    :information_source: *Device Posture overview* {{ .Date.Format "Jan 02, 2006 15:04:05 UTC" }}

    {{ if and (not .Falcon) (not .WS1) }}Nothing to report!  :white_check_mark: {{ else }}

    {{ range $result := .Falcon }}
    :man-surfing: *{{ $result.Email }}*
    {{ range $device := $result.Devices }}
      :computer: {{ $device.MachineName}}
      {{ range $vuln := $device.Findings }}- {{ $vuln.ProductName }} ({{ $vuln.CveSeverity }}) (Open for {{ $vuln.DaysOpen }} days) ({{ $vuln.CveID }}){{ end }}
    {{ end }}
    {{ end }}

    {{ range $result := .WS1 }}
    :man-surfing: *{{ $result.Email }}*
    {{ range $device := $result.Devices }}
      :computer: {{ $device.MachineName }}
      Compromised: {{ $device.Compromised }}
      Last seen: {{ $device.LastSeen.Format "Jan 02, 2006 15:04:05 UTC" }}
      {{ range $finding := $device.Findings }}- :warning: {{ $finding.ComplianceName }}{{ end }}
    {{ end }}
    {{ end }}
    {{ end }}

    {{ if .Errors }}
    :warning: *Errors:*
    {{ range $err := .Errors }}
    - {{ $err }}
    {{ end }}
    {{ end }}
```
7. Run `css -config=your-config.yml -log=debug -dry` to test.
8. See the security overview popup to you in Slack!
9. Now run it for real with `css -config=your-config.yml`.
