---
layout: page
title: Request Signing Examples
--- 

# Examples
## Sign Request
Method to generate Signature
``` csharp
private string GenerateSignature(HttpMethod method, string date, string httpResource)
{
    var stringToSign = method + "\n" +
                       //"ToLowerCase(Content-MD5)" +
                       "\n" +
                       "application/json" +
                       "\n" +
                       date + "\n" +
                       httpResource;
    return HmacSha1(_secretAccessKey, stringToSign);
}
```

Method to generate Hashed string
``` csharp
 private string HmacSha1(string key, string dataToSign)
{
    Byte[] secretBytes = Encoding.UTF8.GetBytes(key);
    HMACSHA1 hmac = new HMACSHA1(secretBytes);
    Byte[] dataBytes = Encoding.UTF8.GetBytes(dataToSign);
    Byte[] calcHash = hmac.ComputeHash(dataBytes);
    String calcHashString = Convert.ToBase64String(calcHash);
    return calcHashString;
}
```
Example API call with signed request header
``` csharp
public ShipmentResponse GenerateShippingLabel(int orderId, int kitsQty, string contactFullName, string zip, string city,
    string address1, string address2, string email, string phone)
{
    var shipmentModel = new Shipment()
    {
        Pieces = new[]
        {
            GeneratePieceByKitsQty(kitsQty)
        },
        ReadyByDate = DateTime.Now.AddMinutes(10),
        Hawb = orderId.ToString(),
        Description = "Kits for medical tests",
        NonDox = true,
        DDP = true,
        Pallet = false,
        Consignee = new Consignee
        {
            ContactName = contactFullName,
            CountryCode = "GB",
            Zipcode = zip,
            City = city,
            Address1 = address1,
            Address2 = address2,
            Email = CommonHelper.IsValidEmail(email) ? email : null,
            PhoneNumber = phone
        },
        Service = new Service
        {
            Code = "1CP"
        },
        LabelFormat = "pdf"
    };

    string dateString = DateTime.Now.ToString("R");
    string httpResource = "/api/shipment";

    using (var requestMessage =
        new HttpRequestMessage(HttpMethod.Post, httpResource))
    {
        requestMessage.Headers.TryAddWithoutValidation(HeaderNames.Authorization,
            $"{_accessKeyId}:{GenerateSignature(HttpMethod.Post, dateString, $"{httpResource}")}");
        requestMessage.Headers.Add(HeaderNames.Date, dateString);

        requestMessage.Content = new JsonContent(JsonConvert.SerializeObject(shipmentModel));

        var response = _httpClient.SendAsync(requestMessage).ContinueWith(task =>
        {
            try
            {
                return task.Result;
            }
            catch (AggregateException ex)
            {
                return null;
            }
            catch (WebException ex)
            {
                return null;
            }
        }).Result;

        if (response == null)
        {
            return null;
        }

        var responseText = response.Content.ReadAsStringAsync().Result;
        _logger.InsertLog(LogLevel.Debug, "Norsk Response: Create Shipment", responseText);

        var shipmentResponse = JsonConvert.DeserializeObject<ShipmentResponse>(responseText);
        if (shipmentResponse != null)
        {
            return shipmentResponse;
        }
    }


    return null;
}

```