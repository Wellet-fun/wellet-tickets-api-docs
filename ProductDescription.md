# Product Description Definition

| Parameter      | Type    | Description                                                                                                           |
|----------------|---------|-----------------------------------------------------------------------------------------------------------------------|
| id             | integer | Product identifier                                                                                                    |
| name           | string  | Product name                                                                                                          |
| description    | string  | Product description                                                                                                   |
| features       | array   | Array of strings that detail what is included with this product                                                       |
| itemsAvailable | integer | Max number of items that are available for this date and performance                                                  |
| price          | object  | Price detailed in the requested currency                                                                              |
| ageRange       | object  | Details which age range this product is availabe at the given price. For example: adults, less than 12 years old, etc |