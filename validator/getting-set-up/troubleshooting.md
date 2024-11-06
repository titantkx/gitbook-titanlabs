# 🐞 Troubleshooting

<details>

<summary>System permission error (authentication required)</summary>

When configuring the validator node, certain steps necessitate setting up services, which in turn require root permissions.\
E.g. when create/update `.service` file in `/etc/systemd/system/` or commands start with `systemctl`&#x20;

You can get that error if you set up a node under the user account in the linux system. To solve it, you can add `sudo` before commands that require root permission. Like `sudo systemctl daemon-reload`

</details>

