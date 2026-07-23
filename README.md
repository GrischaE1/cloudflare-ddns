# cloudflare-ddns

Cloudflare DDNS for QNAP and Synology NAS and other Linux systems. Based on Cloudflare Worker or other Serverless functions.

With this small interface, it is possible to host your own API to use Cloudflare domains for DynDNS on QNAP or Synology NAS systems.

The API responds with plain text and matching status codes for QNAP and Synology systems.

------------

## Authentication

This Worker supports two Cloudflare authentication methods:

1. **API Token (recommended):** create a token with `Zone > Zone > Read` and `Zone > DNS > Edit` permissions, and limit it to the zone you want to update. Send it in an `Authorization: Bearer <token>` header, or use `api_token` for NAS clients that cannot send headers.
2. **Global API Key (legacy):** supply your Cloudflare account `email` together with `api_key`. This remains available for existing deployments, but it has broad account permissions and is not recommended for new deployments.

See Cloudflare's [API authentication guide](https://developers.cloudflare.com/fundamentals/api/get-started/) for the two authentication schemes.

Required information:

- Cloudflare API Token **or** Cloudflare Account email and [Global API Key](https://dash.cloudflare.com/profile/api-tokens) *(not Origin CA Key)*
- Cloudflare registered domain *(like `example.com`)*
- DNS Record *(like `my-ddns.example.com`)*

------------

## Cloudflare Worker

### Setup

1. Go to [Cloudflare Workers](https://workers.cloudflare.com/) and create a new Worker.
2. Copy the content of `worker.js` into the editor.
3. Click `Save and Deploy`.

## Usage

### API Token (recommended)

Prefer the `Authorization` header so the token is not part of the URL:

```bash
curl -H "Authorization: Bearer YOUR_API_TOKEN" \
  'https://your.cloudflare.worker.host/route/to/worker?record=my-ddns.example.com&ip=YOUR_IP&ttl=120'
```

For NAS clients that cannot send request headers, use `api_token` without the `email` parameter:

```
https://your.cloudflare.worker.host/route/to/worker?api_token=YOUR_API_TOKEN&record=my-ddns.example.com&ip=YOUR_IP&ttl=120
```

> Tokens in URL query strings can be captured by logs. Use a narrowly scoped token, do not share the URL, and rotate the token if it may have been exposed.

### Global API Key (legacy compatibility)

Existing clients can continue using the original `email` and `api_key` parameters:

```
https://your.cloudflare.worker.host/route/to/worker?email=cloudflare@email.com&api_key=YOUR_GLOBAL_API_KEY&record=my-ddns.example.com&ip=YOUR_IP&ttl=120
```

### DynDNS for QNAP NAS

`Network- and Virtual Switch` -> `DDNS` -> `Add` -> `Select DNS server: Customized`

![QNAP DDNS](https://raw.githubusercontent.com/lmxx1234567/cloudflare-ddns/main/images/qnap-ddns.png "QNAP DDNS")

For the recommended API Token configuration:

- Username: any non-empty value (it is not sent to the Worker)
- Password: your Cloudflare API Token
- Hostname: your DNS Record *(like `my-ddns.example.com`)*
- URL:

```
https://your.cloudflare.worker.host/route/to/worker?api_token=%PASS%&record=%HOST%&ip=%IP%&ttl=120
```

For Global API Key compatibility, set Username to your Cloudflare Account email, Password to your Global API Key, and use:

```
https://your.cloudflare.worker.host/route/to/worker?email=%USER%&api_key=%PASS%&record=%HOST%&ip=%IP%&ttl=120
```

------------

### DynDNS for Synology NAS

`System Controls` -> `External Access` -> `Customize`

![Synology DDNS Provider](https://raw.githubusercontent.com/fbrettnich/cloudflare-dyndns-php/main/.github/images/synology-ddns-provider.png "Synology DDNS Provider")

`System Controls` -> `External Access` -> `Add`

![Synology DDNS](https://raw.githubusercontent.com/fbrettnich/cloudflare-dyndns-php/main/.github/images/synology-ddns.png "Synology DDNS")

API Token URL:

```
https://your.cloudflare.worker.host/route/to/worker?api_token=__PASSWORD__&record=__HOSTNAME__&ip=__MYIP__&ttl=120
```

Legacy Global API Key URL:

```
https://your.cloudflare.worker.host/route/to/worker?email=__USERNAME__&api_key=__PASSWORD__&record=__HOSTNAME__&ip=__MYIP__&ttl=120
```

------------

### DynDNS for Linux

API Token cURL command:

```bash
curl 'https://your.cloudflare.worker.host/route/to/worker?api_token=YOUR_API_TOKEN&record=my-ddns.example.com&ip=$(curl -s https://ipinfo.io/ip)&ttl=120'
```

Legacy Global API Key cURL command:

```bash
curl 'https://your.cloudflare.worker.host/route/to/worker?email=cloudflare@email.com&api_key=XXXX&record=my-ddns.example.com&ip=$(curl -s https://ipinfo.io/ip)&ttl=120'
```

To get your public IP address, you can use:

```bash
curl https://ipinfo.io/ip
```

## TODO

- [x] Add support for custom DNS settings like `proxied` or `auto_ttl`
- [ ] Add support for automatic SSL certificate renewal

## License

[MIT](LICENSE)

## Credits

This project is inspired by [fbrettnich/cloudflare-dyndns-php](https://github.com/fbrettnich/cloudflare-dyndns-php). We ported the PHP code to Cloudflare Worker to make it more scalable and easier to use.
