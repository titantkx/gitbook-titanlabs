# Testnet Faucet Integration

If you want to have a testnet faucet integration within your dApp for the Titan network, the only thing you need to do is make a `POST` request to `https://titan-testnet-faucet.titanlab.io/api/v1` and pass an `{ address: titan1...}` as the body of the `POST` request. The address is then stored within the queue, which is processed every 5 to 10 minutes.

## API Endpoint

* **URL**: `https://titan-testnet-faucet.titanlab.io/api/v1`
* **Method**: `POST`
* **Content-Type**: `application/json`

## Request Body

```json
{
  "address": "titan1..."
}
```

## Example Implementation

Here is an example code snippet:

```typescript
// Using fetch
const requestFaucet = async (address: string) => {
  try {
    const response = await fetch(
      "https://titan-testnet-faucet.titanlab.io/api/v1",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ address }),
      }
    );

    if (response.ok) {
      alert("Success! Your address has been added to the queue");
    } else {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
  } catch (error) {
    alert("Something went wrong - " + error.message);
  }
};

// Usage
requestFaucet("titan1....");
```

```typescript
// Using axios
import axios from "axios";

const requestFaucet = async (address: string) => {
  try {
    await axios.post("https://titan-testnet-faucet.titanlab.io/api/v1", {
      address: address,
    });
    alert("Success! Your address has been added to the queue");
  } catch (error) {
    alert("Something went wrong - " + error.message);
  }
};

// Usage
requestFaucet("titan1....");
```

## Important Notes

* **Processing Time**: Requests are processed every 5 to 10 minutes
* **Address Format**: Ensure you're using the correct Titan network address format (`titan1...`)
* **Rate Limiting**: Be mindful of potential rate limiting on the faucet endpoint
* **Error Handling**: Always implement proper error handling for network requests
