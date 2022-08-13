# Homebrew push notifications without libraries.

## Listen

### Windows

```powershell
$HttpWebRequest = [System.Net.HttpWebRequest]::Create('https://rppico-default-rtdb.asia-southeast1.firebasedatabase.app/msg.json')
$HttpWebRequest.Accept = 'text/event-stream'
$ResponseStream = $HttpWebRequest.GetResponse().GetResponseStream()
$StreamReader = [System.IO.StreamReader]::new($ResponseStream)

while (($line = $StreamReader.ReadLine()) -ne $null)
{    
    if (-! $line.StartsWith('data: {')){ continue }
    $ToastText01 = [Windows.UI.Notifications.ToastTemplateType, Windows.UI.Notifications, ContentType = WindowsRuntime]::ToastText01
    $TemplateContent = [Windows.UI.Notifications.ToastNotificationManager,Windows.UI.Notifications, ContentType = WindowsRuntime]::GetTemplateContent($ToastText01)
    $TemplateContent.SelectSingleNode('//text[@id="1"]').InnerText = ($line.Substring(6) | ConvertFrom-Json).data
    $AppId = '{1AC14E77-02E7-4E5D-B744-2EB1AE5198B7}\WindowsPowerShell\v1.0\powershell.exe'
    [Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier($AppId).Show($TemplateContent)
}

$StreamReader.Close() > $null
$ResponseStream.Close() > $null
```

### curl

```bash
curl https://rppico-default-rtdb.asia-southeast1.firebasedatabase.app/msg.json -H Accept:text/event-stream --no-buffer \
  | grep '^data: {' --line-buffered \
  | while read line; do notify-send "${line:26:-2}"; done
```

### wget

```bash
wget --header Accept:text/event-stream -qO- https://rppico-default-rtdb.asia-southeast1.firebasedatabase.app/msg.json \
  | grep '^data: {' --line-buffered \
  | while read line; do notify-send "${line:26:-2}"; done
```

## Publish

### Windows

```powershell
Invoke-WebRequest -Method PUT -Body ([System.Text.Encoding]::UTF8.GetBytes("`"🕊$(date -Format yyyy-MM-ddTHH:mm:ss.fff)`"")) https://rppico-default-rtdb.asia-southeast1.firebasedatabase.app/msg.json
```

### curl

```bash
curl -X PUT -d '"🐥'$(date --iso-8601=ns)'"' https://rppico-default-rtdb.asia-southeast1.firebasedatabase.app/msg.json
```

### wget

```bash
wget --method PUT --body-data '"🐧'$(date --iso-8601=ns)'"' -qO- https://rppico-default-rtdb.asia-southeast1.firebasedatabase.app/msg.json
```

## Endpoint

Open Firebase console and copy url

![Screenshot from 2022-08-14 02-15-45](https://user-images.githubusercontent.com/12811398/184504259-5bf12e7a-c2c4-438c-af09-52632830ff15.jpg)


# References

https://firebase.google.com/docs/reference/rest/database#section-streaming
