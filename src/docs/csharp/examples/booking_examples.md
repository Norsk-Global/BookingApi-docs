---
layout: page
title: Booking Examples
--- 

# Examples
## Book Shipment

Booking a shipment using static volatile ApiClient.
``` csharp
var response = await ApiClient
                .BookShipment(booking => booking
                    .CreateBooking()
                    .WithReadyByDate()
                    .WithHawb("123123123")
                    .WithDescription("test Clothing")
                    .WithDocuments()
                    .WithPieces(pieces => pieces
                        .AddPiece(piece => piece
                            .Depth(1.00m)
                            .Height(1.00m)
                            .Width(1.00m)
                            .Weight(1.00m)
                            .NumberOfPieces(1)))
                    .WithConsignee(consignee => consignee
                        .ContactName("Test")
                        .Company("Test")
                        .Address1("2 Willow Road")
                        .Zipcode("SL3 0BS")
                        .City("Slough")
                        .CountryCode("GB"))
                    .WithServiceCode("IEL")
                );
```

Booking a shipment using the Dependency Injection
``` csharp
    public class BookingPipeline
    {
        private readonly IApiClient _apiClient;

        public BookingPipeline(IApiClient apiClient)
        {
            _apiClient = apiClient;
        }

        public async Task<string> BookShipment()
        {
            var response = await _apiClient
                            .BookShipment(booking => booking
                                .CreateBooking()
                                .WithReadyByDate()
                                .WithHawb("123123123")
                                .WithDescription("test Clothing")
                                .WithDocuments()
                                .WithPieces(pieces => pieces
                                    .AddPiece(piece => piece
                                        .Depth(1.00m)
                                        .Height(1.00m)
                                        .Width(1.00m)
                                        .Weight(1.00m)
                                        .NumberOfPieces(1)))
                                .WithConsignee(consignee => consignee
                                    .ContactName("Test")
                                    .Company("Test")
                                    .Address1("2 Willow Road")
                                    .Zipcode("SL3 0BS")
                                    .City("Slough")
                                    .CountryCode("GB"))
                                .WithServiceCode("IEL")
                            );
            
            return response.Barcode;
        }
    }
```