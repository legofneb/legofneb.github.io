---
layout: post
title:  "Connecting from PHP to Window's SOAP Services"
date:   2016-03-06
categories: PHP, SOAP
---

I was running into an issue connecting from PHP's soap client to an IIS hosted SOAP service. The resulting XML from PHP was valid SOAP XML, but kept getting rejected by the service.

Here is the general structure of the XML:

```
<soap:Envelope>
  <soap:Header>
    <soap:UserInfo>
      <userid></userid>
      <password></password>
    </soap:UserInfo>
  </soap:Header>
  <soap:Body>
    <CreateRequest>
      <params>a serialized string of parameters representing the request</params>
    </CreateRequest>
  </soap:Body>
</soap:Envelope>
```

This should be easy enough, here is the PHP to create the request:

```
$client = new SoapClient('https://soap.service/Controller.asmx?WSDL', array());
$namespace = 'https://my.ns';
$userInfoHeader = new SOAPHeader($namespace, 'UserInfo', array('userid' => 'user', 'password' => 'pw'), false);
$client->__setSoapHeaders($userInfoHeader);
$client->CreateRequest(array('params' => 'some parameters to send to server'));

```

But I kept getting a (400) Bad Request from the server. Since I have no control over the SOAP Service, I had to alter my XML to match their specifications. The documentation had been given in C# and VB, and was intended for windows machines, so I just had to edit the XML to match that of the windows.

After reading through some documentation, and some stackoverflow, I think the best spot would be to alter the XML right before the request and transform/change it to whatever I needed. This is what I found to work:

```
$namespace = 'https://my.ns';

class CustomSoapClient extends SoapClient {
  function __doRequest($request, $location, $action, $version, $one_way = 0) {
    global $namespace;

    // Changing soap tag prefix
    $request = str_replace( 'SOAP-ENV', 'soap', $request );

    // Removing namespace prefixes
    $request = str_replace( '<ns1:', '<', $request );
    $request = str_replace( '</ns1:', '</', $request );
    $request = str_replace( ' xmlns:ns1="' . $namespace . '"', '', $request );

    // Only using namespaces on functions
    $request = str_replace( '<CreateRequest', '<CreateRequest xmlns="' . $namespace . '"', $request );

    // matching c# client envelope namespaces
    $request = str_replace( 'xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"', 'xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"', $request);

    return parent::__doRequest($request, $location, $action, $version, $one_way);
  }  
}

$client = new CustomSoapClient('https://soap.service/Controller.asmx?WSDL', array());
$namespace = 'https://my.ns';
$userInfoHeader = new SOAPHeader($namespace, 'UserInfo', array('userid' => 'user', 'password' => 'pw'), false);
$client->__setSoapHeaders($userInfoHeader);
$client->CreateRequest(array('params' => 'some parameters to send to server'));

```

So that's it. It work! Now this is a very tailored solution to my problem, where my XML is small, and the problems were limited to a small set of tags. This could definitely use some refactoring to be able to scale to larger service calls.
