# IKEv2 VPN Server on Docker

Recipe to build [`libardoram/ikev2-vpn-server`](https://hub.docker.com/r/libardoram/docker-ikev2-vpn-server/) Docker image.

Taken from build [`gaomd/ikev2-vpn-server`](https://registry.hub.docker.com/u/gaomd/ikev2-vpn-server/) Docker image.

## Usage

### 1. Start the IKEv2 VPN Server

The container requires a `tmpfs` mount at `/run/secrets` so the shared secret is held only
in memory and is never written to disk. A new secret is generated on every container start.

    docker run -d \
      --name ikev2-vpn-server \
      --restart=always \
      --privileged \
      --mount type=tmpfs,destination=/run/secrets \
      -p 500:500/udp \
      -p 4500:4500/udp \
      libardoram/docker-ikev2-vpn-server:latest

### 2. Generate the .mobileconfig (for iOS / OS X)

Because the shared secret lives only in memory (tmpfs), profile generation must run inside
the already-running container using `docker exec`:

    docker exec ikev2-vpn-server generate-mobileconfig > ikev2-vpn.mobileconfig

Transfer the generated `ikev2-vpn.mobileconfig` file to your local computer via SSH tunnel
(`scp`) or any other secure method.

> **Note:** The shared secret is regenerated every time the container restarts. You must
> re-run the command above and redistribute the `.mobileconfig` to all clients after each
> restart.

### 3. Install the .mobileconfig (for iOS / OS X)

- **iOS 9 or later**: AirDrop the `.mobileconfig` file to your iOS 9 device, finish the **Install Profile** screen;

- **OS X 10.11 El Capitan or later**: Double click the `.mobileconfig` file to start the *profile installation* wizard.

## Published Image

| Tag | Digest |
|---|---|
| `noble-20260113` | `sha256:453b9fab00522851fc628b7967c65571b963ed4657d08389e5bac4d081b7194a` |
| `latest` | same as above |

## Technical Details

Upon container start, a *shared secret* is randomly generated using `openssl rand` and written
to `/run/secrets/ipsec.secrets`, which is backed by a `tmpfs` mount — it exists only in memory
and is never persisted to disk. `/etc/ipsec.secrets` in the image contains only an `include`
directive pointing to that path. No *certificate*, *username*, or *password* is used.

Because the secret is ephemeral, any existing client connections will break if the container
is restarted. Regenerate and redistribute the `.mobileconfig` after every restart.

## License

Copyright (c) 2016 Mengdi Gao, This software is licensed under the [MIT License](LICENSE).

---

\* IKEv2 protocol requires iOS 8 or later, Mac OS X 10.11 El Capitan is supported as well.

\* Install for **iOS 8 or later** or when your AirDrop fails: Send an E-mail to your iOS device with the `.mobileconfig` file as attachment, then tap the attachment to bring up and finish the **Install Profile** screen.
