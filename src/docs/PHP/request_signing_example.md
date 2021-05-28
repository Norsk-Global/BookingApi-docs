---
layout: page
title: Request Signing Examples
---


# Examples
## Signing Request to Norsk API

``` PHP
<?php

$NorskAccessKeyId="";
$NorskSecretAccessKey="";
date_default_timezone_set('Europe/London');
$d = time();
$now = date('D, d M Y H:i:s', $d);
$contentType="application/json";
$body =''; // request body

$md5=md5($body, false);
$StringToSign = "POST\n".$md5."\n".$contentType."\n".$now." GMT\n"."/api/shipment";
$Signature = Base64_encode(hash_hmac("SHA1", $StringToSign,$NorskSecretAccessKey,true));

echo $StringToSign, "\n";

$url = 'http://api.norsk-global.com/api/shipment';

// use key 'http' even if you send the request to https://...
$options = array(
    'http' => array(
        'ignore_errors' => true,
        'header' => 'Authorization: '.$NorskAccessKeyId.":".$Signature."\r\n".
            'Date: '.$now."\r\n".
            "Accept: application/json\r\n".
            "Content-Type: application/json\r\n",
        'method'  => 'POST',
        'content' => $body
    )
);

$context  = stream_context_create($options);
$result = @file_get_contents($url, false, $context);
if ($result === FALSE) {
    echo 'Errror';
}
// This will echo the results, you might get {"Message":"An unknown error occurred","Reference":"11b64e965ac34bd8a9c34ae17cddc195"}
// In which case just pass over the reference to us so we can investigate the issue easier.

echo $result
?>
```