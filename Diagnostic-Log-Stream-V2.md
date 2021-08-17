# Overview
The endpoint /logstreamV2 enables you to get the logs emitted by your application using a non-persistent connection. It is similar to [Diagnostic Log Stream](https://github.com/projectkudu/kudu/wiki/Diagnostic-Log-Stream), but also different in some ways.

# Usage
The simplest way to get the logs would be to use curl. 
```
curl -u {username} https://{sitename}.scm.azurewebsites.net/api/logstreamV2
```
Here `username` is your Azure Publishing user (same as you use for git publishing). The command will prompt for your password.

Please refer notes from [Diagnostic Log Stream](https://github.com/projectkudu/kudu/wiki/Diagnostic-Log-Stream) for how to use curl and other details.

# Optional Parameters
There are a few parameters which you can use to control what is returned by the API endpoint.
* **max** : Use this parameter to get a maximum of 'max' number of log entries. The default value is 50.
* **nextToken** : This parameter lets you continue reading from where the previous call stopped. It is also returned with every call and can be used directly with the next call.

**Examples** 
To get a maximum of 100 entries use -
```
curl -u {username} https://{sitename}.scm.azurewebsites.net/api/logstreamV2?max=100
```
To get the next 100 entries use -
```
curl -u {username} https://{sitename}.scm.azurewebsites.net/api/logstreamV2?max=100&nextToken={nextToken}
```
where the `{nextToken}` is the value in the nextToken field returned by the first call. You can similarly call the end point again using value in the nextToken field to get the next 'max' number of log entries.

# Return Value
The end point returns in the following format - 
```
{
  {"nextToken": {nextToken}},
  {"entries": [
    { "--logEntry1--" },
    { "--logEntry2--" },
    ..
    { "--logEntryMax--"}]
  }
}
```
The log file rolls over when the size of it reaches a certain size. If that happens, the `nextToken` field is ignored and the reading starts from the start. The output would be in the following format -
```
{
  {"nextToken": {nextToken}},
  {"entries": [
    {Log file has rolled over.}
    { "--logEntry1--" },
    { "--logEntry2--" },
    ..
    { "--logEntryMax--"}]
  }
}
```
