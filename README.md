# Crisphive Java

The official Java SDK for the
[Crisphive API](https://docs.crisphive.com/).

Typed access to the public `/v1` API — customers, bookings, catalog, team and
fleet.

## Requirements

Java 8+. Maven or Gradle.

## Installation

Maven:

```xml
<dependency>
  <groupId>com.crisphive</groupId>
  <artifactId>crisphive-java</artifactId>
  <version>0.1.0</version>
</dependency>
```

Gradle:

```groovy
implementation "com.crisphive:crisphive-java:0.1.0"
```

## Authentication

Every request is authenticated with a secret API key sent as a bearer token.
Create keys from your Crisphive business dashboard. **The key prefix selects the
data environment:**

- `chsk_live_…` → live (production) data
- `chsk_test_…` → sandbox (isolated test) data

Load keys from the environment — never commit them.

## Usage

```java
import com.crisphive.client.ApiClient;
import com.crisphive.client.ApiException;
import com.crisphive.client.Configuration;
import com.crisphive.client.auth.HttpBearerAuth;
import com.crisphive.client.api.CustomerApi;
import com.crisphive.client.model.*;

public class Example {
  public static void main(String[] args) throws Exception {
    ApiClient client = Configuration.getDefaultApiClient();

    // SDK adds the "Bearer " prefix.
    HttpBearerAuth apiKeyAuth = (HttpBearerAuth) client.getAuthentication("ApiKeyAuth");
    apiKeyAuth.setBearerToken(System.getenv("CRISPHIVE_API_KEY"));

    CustomerApi customers = new CustomerApi(client);

    // List customers.
    ListCustomers200Response res = customers.listCustomers(null, null, null, null, null, 1, 20, null);
    if (res.getData() != null && res.getData().getCustomers() != null) {
      res.getData().getCustomers().forEach(c ->
          System.out.println(c.getId() + " " + c.getFullName()));
    }

    // Create a customer.
    try {
      CustomerCreateRequest body = new CustomerCreateRequest()
          .fullName("Ada Lovelace").email("ada@example.com");
      CreateCustomer200Response created = customers.createCustomer(body, null);
      System.out.println("created " + created.getData().getCustomerId());
    } catch (ApiException e) {
      System.err.println("error " + e.getCode() + " " + e.getResponseBody());
    }
  }
}
```

> Method parameter lists follow the generated signatures — see the [API reference](https://docs.crisphive.com/technical-reference)
> for the exact arguments of each call.

## Pagination

List methods accept `page` / `limit` and return a `meta` object (`total`,
`count`, `per_page`, `current_page`, `total_pages`).

## Idempotency

`create*` calls (customers, bookings) accept an `Idempotency-Key` so retries
never create a duplicate.

## Errors

Non-2xx responses throw `ApiException`; inspect `e.getCode()` and
`e.getResponseBody()` for the Crisphive error code.

## Documentation

- Docs: https://docs.crisphive.com
- API reference: https://docs.crisphive.com/technical-reference
- Webhooks: https://docs.crisphive.com/webhook

## License

[MIT](LICENSE)
