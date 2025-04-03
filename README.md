# zotdav

Easily setup an HTTPS enabled, nginx-based WebDAV server for Zotero using
Docker. This project includes fetching TLS certificates using certbot, and
extended support for WebDAV using the `nginx-dav-ext` module.

## Setup WebDAV

1.  Store credentials in a `.env` file.

    ```bash
    # .env
    WEBDAV_USER=username
    WEBDAV_PASS=password
    ```

2.  Configure mount paths on your host.

    ```yaml
    services:
    # ...
    zotdav:
      # ...
      volumes:
        - "/path/to/webdav:/var/www/webdav"
    ```

    Make sure `/path/to/webdav` exists on the host.

3.  Run the service using docker compose.

    ```console
    $ docker compose up -d
    ```

    To follow the logs, you can use `docker compose logs -f`

## Configure Nginx Proxy Manager

Add a new proxy host with the following details

    Forward Hostname: zotdav
    Forward Port: 8000 

## Setup Nginx Proxy Manager

We shall use Nginx Proxy Manager (NPM) as a frontend for our WebDAV service. The
biggest advantage is that NPM manages certificates for you. It also allows you
to easlily setup multiple services on your server. We shall use the following
steps to make sure that NPM and zotdav are on the same network. Assume NPM's
default network is called `proxyman_default`.

1.  Follow the setup instructions listed on NPM's
    [website](https://nginxproxymanager.com/setup/)
2.  Add `proxyman_default` to the networks section of the zotdav service.
3.  You may also need to add a `default` network along with `proxyman_default`.
4.  Mark `proxyman_default` as an external network in the compose file.
5.  Remove any ports that were previously exposed from the service.
6.  On the NPM web interface, add a proxy host using `zotdav` as the forward
    hostname and 8080 as the forward port.

```
services:
  immich-server:
    .....
    # ports:
    #   - 2283:3001
    networks:
      - proxyman_default

networks:
  proxyman_default:
    external: true
```

## Setup Zotero

1. Open Zotero settings. In the "Sync" section, select "Sync attachment files in
   My Library using WebDAV".
2. Enter the URL for your WebDAV server. Typically this will also your certbot
   domain.
3. Enter the username and password for your WebDAV server. You set these in the
   `.env` file.
4. Zotero will automatically append `/zotero/` to the URL you enter. If you need
   a different path, you will need to make sure that the path exists on the
   filesystem.
5. For reference, `https://yourdomain.com/webdav/path/to/zotero` will correspond
   to `/path/to/webdav/path/to/zotero` on your filesystem.

## Footnotes

1. Certbot sets up a cron job to renew certificates automatically. Certbot logs
   can be found at `/var/log/letsencrypt/letsencrypt.log`.
1. Install the `nginx-dav-ext` module using `apt install nginx-extras`. After
   installation, `nginx-dav-ext` needs to be loaded in `nginx.conf`.
1. While fetching certificate for the first time, `nginx.conf` should be valid.
   This means there cannot be any TLS servers in `nginx.conf` when certbot is
   running. This is also why this project has two configs for nginx.
1. While HTTPS is not required for WebDAV, Zotero's iPad app will refuse to
   authenticate over HTTP.
