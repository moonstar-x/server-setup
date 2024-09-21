# Introduction

This is a simple guide that focuses on the installation and configuration process of my self hosted infrastructure that runs multiple services for various purposes. The server now uses exclusively Docker for all of its services.

This is an improved version of the [previous server guide](https://moonstar-x.dev/old-server-setup), some of the services from the previous version are present here, some are gone for good.

As you will notice, this server setup is actually composed of multiple devices with specific roles. That go along the lines of:

> * [Alpha (Home Server)](./servers/alpha/setting-up/index.md)
>
> Hosts most of the services I host from my own self-built machine.
>
> * [Bravo (VPS)](./servers/bravo/setting-up/index.md)
>
> Hosts some of the services that need a higher availability than my home server can provide.
>
> * [Delta (VPS)](./servers/delta/setting-up/index.md)
>
> Hosts my personal Discord bot deployments.
