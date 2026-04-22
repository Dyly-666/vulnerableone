---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/reconnaissance/search-victim-owned-websites
---

# Search Victim-Owned Websites

Adversaries may search websites owned by the victim for information that can be used during targeting. Victim-owned websites may contain a variety of details, including names of departments/divisions, physical locations, and data about key employees such as names, roles, and contact info. These sites may also have details highlighting business operations and relationships.[\[1\]](https://www.comparitech.com/blog/vpn-privacy/350-million-customer-records-exposed-online/)

## Extract Link From HTML

```basic
cat index.html | sed 's/href=/\nhref=/g' | \grep href=\" | \sed 's/.*href="//g;s/".*//g' | sort | uniq
cat index.html | sed 's/src=/\nsrc=/g' | \grep src=\" | \sed 's/.*src="//g;s/".*//g' | sort | uniq
cat index.html | grep -Po '<img[^>]*src="\K[^"]*(?=")'
```

{% code title="Grep From Site" %}
```powershell
# Specify the URL of the web page you want to extract attributes from
$url = "https://example.com"

# Use Invoke-WebRequest to fetch the web page content
$response = Invoke-WebRequest -Uri $url

# Check if the request was successful
if ($response.StatusCode -eq 200) {
    # Extract src and href attributes using regular expressions
    $htmlContent = $response.Content
    $srcMatches = [System.Text.RegularExpressions.Regex]::Matches($htmlContent, 'src="([^"]+)"')
    $hrefMatches = [System.Text.RegularExpressions.Regex]::Matches($htmlContent, 'href="([^"]+)"')

    # Print the extracted src attributes
    Write-Host "SRC attributes:"
    foreach ($match in $srcMatches) {
        Write-Host $match.Groups[1].Value
    }

    # Print the extracted href attributes
    Write-Host "HREF attributes:"
    foreach ($match in $hrefMatches) {
        Write-Host $match.Groups[1].Value
    }
} else {
    Write-Host "Failed to retrieve the web page. Status code: $($response.StatusCode)"
}
```
{% endcode %}
