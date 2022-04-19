# Wellet Tickets Api
This repository contains documentation for using [wellet.fun](https://wellet.fun/) api for obtaining information and booking tickets from shows in the Wellet platform.

## Environments
Wellet provides a production environment and a sandbox environment. The sandbox environment allows developers to integrate with the Wellet platform without affecting critical data. All the functionality for obtaining products and booking orders are fully implemented in both environments. Sandbox is important for testing the functionality and logic of third party applications before deploying to production or releasing to customers.

## Base URLs

| Environment |          Base URL         |
|:-----------:|:-------------------------:|
| Production  | https://api.wellet.fun    |
| Sandbox     | https://apidev.wellet.fun |

## Api Key

All the calls to this api need to specify an api key, in the `x-api-key` http header. The api key will be provided by the Wellet team. Contact [info@wellet.fun](info@wellet.fun) if you want to integrate with the wellet platform.

## Endpoints

All endpoints are detailed in the [swagger documentation](https://docs.wellet.fun/).

## Typical workflow

The following endpoints are listed in the typical order for a booking ticket workflow.

### 1) Get Shows
Returns a list of shows enabled for sale. 

#### Endpoint
```
GET /shows
```

#### Input Parameters
None

#### Output Parameters
Returns an array of shows, each show containing the following properties:

| Parameter | Type    | Description                                            |
|-----------|---------|--------------------------------------------------------|
| id        | integer | Show identifier                                        |
| name      | string  | Name of the show                                       |
| image     | string  | Url pointing to the show promotional image             |
| enabled   | boolean | True if show is enabled for sale                       |
| venue     | object  | Object describing the venue where the show takes place |

### Example

Example request:
```
curl --location --request GET 'https://apidev.wellet.fun/shows/' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:
```
[
    {
        "id": 123,
        "name": "Cool show",
        "image": "https://cdn1.wellet.fun/some-image",
        "enabled": true,
        "venue": {
            "name": "Cool venue",
            "address": "Blvd. Kukulcan Km 1 #123; 77500 QR",
            "postalCode": "77500",
            "city": "Cancún",
            "state": "Quintana Roo",
            "country": "Mexico",
            "latitude": 21.62,
            "longitude": -86.84,
            "phone": "123456789"
        }
    },
    {
        "id": 124,
        "name": "Another cool show",
        "image": "https://cdn1.wellet.fun/another-image",
        "enabled": true,
        "venue": {
            "name": "Cool venue 2",
            "address": "Blvd. Kukulcan Km 1 #123; 77500 QR",
            "postalCode": "77500",
            "city": "Cancún",
            "state": "Quintana Roo",
            "country": "Mexico",
            "latitude": 21.62,
            "longitude": -86.84,
            "phone": "123456789"
        }
    }
]
```

### 2) Get Products for a particular show and date
Returns a list of `Performances` with its corresponding products offered by the show in that particular date. A performance is a particular instance of a show in a given day. There may be more than one performance in a given date (for example: the 4 PM performance and the 8 PM performance). 

#### Endpoint
```
GET /shows/{showId}/products
```

#### Input Parameters
| Parameter | Location     | Type    | Description                                                                           |
|-----------|--------------|---------|---------------------------------------------------------------------------------------|
| showId    | Path         | integer | Show identifier                                                                       |
| currency  | Query String | string  | Currency in which prices will be returned. Valid values are: "MXN" and "USD"          |
| lang      | Query String | string  | Language in which product descriptions will be returned. Valid values are: "en", "es" |
| date      | Query String | string  | Date for which products will be returned. Format: "yyyy-mm-dd"                        |

#### Output Parameters
Returns an array of performances, each of them containing:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| id           | integer | Performance identifier                    |
| time         | string  | Local time of show                        |
| timeCheckiIn | string  | Time at which passengers must check in    |
| timeStart    | string  | Time at which show starts                 |
| timeEnd      | string  | Time at which show ends                   |
| products     | array   | Array of products (see description below) |

Each product object contains the following fields:

| Parameter    | Type    | Description                               |
|--------------|---------|-------------------------------------------|
| id           | integer | Performance identifier                    |
| time         | string  | Local time of show                        |
| timeCheckiIn | string  | Time at which passengers must check in    |
| timeStart    | string  | Time at which show starts                 |
| timeEnd      | string  | Time at which show ends                   |
| products     | array   | Array of products (see [description](./ProductDescription.md))       |

### Example

Example request:
```
curl --location --request GET 'https://apidev.wellet.fun/shows/1234/products?currency=mxn&lang=es&date=2022-04-14' \
--header 'x-api-key: YOUR_API_KEY'
```

Example response:

```
{
    "performances": [
        {
            "id": 44,
            "time": "20:00:00",
            "timeCheckIn": "7:00 pm",
            "timeStart": "8:00 pm",
            "timeEnd": "1:00 am",
            "products": [
                {
                    "id": 12,
                    "name": "Some ticket",
                    "description": "Best ticket for dancing",
                    "features": [
                        "Entrada",
                        "Shows",
                        "15 Bebidas nacionales"
                    ],
                    "itemsAvailable": 50,
                    "price": {
                        "amount": 90.0,
                        "currency": "USD"
                    },
                    "ageRange": {
                        "id": 1,
                        "name": "Adultos"
                    }
                },
            ]
        }
    ]
}
```

### 3) Create a new Order for Reservation
An `Order` represents a reservation submitted to the Wellet platform. If the `Order` is approved, the reservation is confirmed with a particular booking code and QR code that needs to be displayed at the Show entrance (either by showing the QR code in a mobile device, or in a printed paper).

#### Endpoint
```
PUT /orders
```

#### Input Parameters
The following parameters are sent in the body payload of the HTTP PUT request:

| Parameter                 | Type    | Description                                                                           |
|---------------------------|---------|---------------------------------------------------------------------------------------|
| showId                    | integer | Show identifier of the show requested to be booked.                                   |
| transactionType           | string  | Type of transaction. Valid values for external users are: "voucher".                  |
| travelVoucher             | string  | Voucher code for this reservation                                                     |
| travelVoucherVerification | string  | Verification code for the voucher (optional)                                          |
| ticketDate                | string  | Date requested to be booked. Format: "yyyy-mm-dd"                                     |
| lang                      | string  | Language to be used for sending the ticket to the customer. Valid values: "en", "es". |
| products                  | array   | Array of products (see [definition](./ProductRequest.md))                                                    |
| customer                  | object  | Customer that will attend to the show (see [definition](./Customer.md))                                |

#### Output Parameters
| Parameter   | Type   | Description                                                                           |
|-------------|--------|---------------------------------------------------------------------------------------|
| bookingCode | string | Booking code that uniquely identifies the reservation                                 |
| qrImage     | string | Url with the QR code that needs to be presented at the show entrance by the customer. |S


### Example

```
curl --location --request PUT 'https://apidev.wellet.fun/orders' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "showId": 2,
    "transactionType": "voucher",
    "travelVoucher": "123456",
    "travelVoucherVerification": "7890123"
    "products": [
        {
            "id": 44,
            "ageRangeId": 1,
            "performanceId": 4,
            "quantity": 2
        },
        {
            "id": 1177,
            "ageRangeId": 1,
            "performanceId": 4,
            "quantity": 2
        }
    ],
    "customer": {
        "fullName": "CUSTOMER NAME",
        "email": "someemail@mail.com",
        "phone": "+1123456789"
    },
    "ticketDate": "2022-04-07",
    "lang": "en",
}'
```

Example response:

```
{
    "bookingCode": "8VX1CD6343",
    "qrImage": "https://apidev.wellet.fun/orders/8VX1CD6343/qr"
}
```

### 4) Cancel Order
An `Order` can be cancelled as long as the passenger has not checked in into the show and cancellation time is appropriate depending on the product type.

#### Endpoint
```
POST /orders/cancel
```

#### Input Parameters
The following parameters are sent in the body payload of the HTTP POST request:

| Parameter                 | Type    | Description                                                                           |
|---------------------------|---------|---------------------------------------------------------------------------------------|
| bookingCode               | string  | Booking code that needs to be cancelled                                               |

#### Output Parameters
| Parameter   | Type   | Description                                                                           |
|-------------|--------|---------------------------------------------------------------------------------------|
| bookingCode | string | Booking code of the reservation                                                       |
| status      | string | New status of the order. Status should be "cancelled" if the request succeeded        |


### Example

```
curl --location --request POST 'https://apidev.wellet.fun/orders/cancel' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "bookingCode": "GLKVUMU1JL" 
}'
```

Example response:

```
{
  "bookingCode": "GLKVUMU1JL",
  "status": "cancelled"
}
```
