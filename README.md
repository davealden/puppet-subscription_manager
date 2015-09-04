# PUPPET-SUBSCRIPTION_MANAGER

This module provides Custom Puppet Provider to handle registration and consumption of RedHat subscriptions using subscription-manager. This module was derived from [puppet-rhnreg_ks module](https://github.com/strider/puppet-rhnreg_ks) by Gaël Chamoulaud.

## License

Read Licence file for more information.

## Requirements
* puppet-boolean [on GitHub](https://github.com/adrienthebo/puppet-boolean)

## Authors
* Gaël Chamoulaud (gchamoul at redhat dot com)
* James Laska (jlaska at redhat dot com)
* JD Powell (waveclaw at hotmail dot com)

## Classes and Defines

This module provides the standard install-configure-service pattern. It also wraps
the provided native resources with a convenience class to enable simple or complex
deployment.

## Examples

Setup to and register one CentOS 6 client to a Katello server

```puppet
class repo::subscription_manager {
# put this in a .pp file somewhere on your Puppet's modulepath
  yumrepo { 'dgoodwin-subscription-manager':
  ensure              => 'present',
  baseurl             => 'https://copr-be.cloud.fedoraproject.org/results/dgoodwin/subscription-manager/epel-6-$basearch/',
  descr               => 'Copr repo for subscription-manager owned by dgoodwin',
  enabled             => '1',
  gpgcheck            => '1',
  gpgkey              =>  'https://copr-be.cloud.fedoraproject.org/results/dgoodwin/subscription-manager/pubkey.gpg',
    skip_if_unavailable => 'True',
  }
}

# put this in either a Profile module or classify in your ENC equivalently
Class { 'subscription_manager':
    repo            => 'repo::subscription_manager',
    server_hostname => 'my_katello.example.com',
    activationkeys  => '1-2-3-example.com-key',
    force           => true,
    org             => 'My_Example_Org',
    server_prefix   => '/rhsm',
    rhsm_baseurl    => 'https://my_katello.example.com/pulp/repos',
  }
}
```
Setup a RedHat Enterprise 7 or CentOS 7 agent to RHN.

```puppet
Class { 'subscription_manager':
   org      => 'My_Company_Org_in_RHN',
   username => 'some_rhn_special_user',
   password => 'password123',
}
```
Not putting the explicit password in the code is a good idea. Using hiera-gpg or hiera-eyaml backends is strongly encouraged for this example.

## Types and Providers

The module adds the following new types:

* `rhsm_register` for managing RedHat Subscriptions
* `rhsm_repo`     for managing RedHat Subscriptions to Repositories
* `rhsm_pool`     for managing RedHat Entitlement Pools (Subscription Collections)

### rhsm_register Parameters

#### Mandatory

- **server_hostname**: Specify a registration server hostname such as subscription.rhn.redhat.com.
- **org**: provide an organization to join (defaults to the Default_Organization
)
On of either the activation key or a username and password combination is needed
to register.  Both cannot be provided and will cause an error.
- **activationkeys**: The activation key to use when registering the system (cannot be used with username and password)
- **password**: The password to use when registering the system
- **username**: The username to use when registering the system

##### Optional

- - **ensure**: Valid values are `present`, `absent`. Default value is `present`.
- **force**: Should the registration be forced. Use this option with caution, setting it true will cause the system to be unregistered before running 'subscription-manager register'. Default value `false`.
- **hardware**: Whether or not the hardware information should be probed. Default value is `true`.
- **pool**: A specific license pool to attach the system to
- **environment**: which environment to join at registration time
- **server_insecure**: If HTTP is used or HTTPS with an untrusted certificate
- **server_prefix**: The subscription path.  Usually /subscription for RHN and /rhsm for a Katello installation.
- **rhsm_baseurl**: The Content base URL in case the registration server has no content. An example would be https://cdn.redhat.com or https://katello.example.com/pulp/repos
- **rhsm_cacert**: Path to a CA certificate for HTTPS connections to the server
- **autosubscribe**: Enable automatic subscription to repositories based on default Pool settings.

### rhsm_register Examples

Register clients to RedHat Subscription Management using an activation key:

<pre>
rhsm_register { 'satelite.example.com':
  server_hostname => 'my-satelite.example.com',
  activationkeys => '1-myactivationkey',
}
</pre>

Register clients to RedHat Subscription management using a username and password:

<pre>
rhsm_register { 'subscription.rhn.example.com':
  username        => 'myusername',
  password        => 'mypassword',
  autosubscribe   => true,
  force           => true,
}
</pre>

Register clients to RedHat Subscription management and attach to a specific license pool:

<pre>
rhsm_register { 'subscription.rhn.example.com':
  username  => 'myusername',
  password  => 'mypassword',
  pool		  => 'mypoolid',
}
</pre>

### rhsm_repo Parameters

If absolutely necessary the individual yum repositories can be filtered.

- **ensure**: Valid values are `present`, `absent`. Default value is `present`.
- **name**: The name of the repository registration to filter.

### rhsm_repo Examples

Add a repo:

<pre>
rhsm_repo { 'rhel-7-server-optional-rpms': }
</pre>

Remove a repo:

<pre>
rhsm_repo { 'rhel-7-server-optional-rpms':
  ensure	  => 'absent',
}
</pre>

### rhsm_pool

Subscriptions to use RHN are sold as either individual entitlements or a pools of
entitlements.  A given server registered to a Satellite 6 or katello system will
consume at least 1 entitlement from a Pool just.

This subscription to the Pool is what enables the set of repositories to be made
available on the server for further subscription.

While this type is mostly useful for exporting the registration information in detail
it can also be used to force switch registrations for selected clients.

### rhsm_pool Parameters
- **name**: Unique Textual description of the Pool
- **ensure**: Is this pool absent or present?
- **provides**: Textual information about the Pool, usually same as the name.
- **sku**: Stockkeeping Unit, usually for inventory tracking
- **account**: Account number for this Pool of Subscriptions
- **contract**: Contract details, if known
- **serial**: Any serial number that is associated with the pool
- **id**: ID Hash of the Pool
- **active**: Is this subscription in use at the moment?
- **quantity_used**: How many is used?  Often licenses are sold by CPU or core so
is it possible for a single server to consume several subscriptions.
- **service_type**: type of service, usually relevant to official RedHat Channels
- **service_level**: level of service such as STANDARD, PREMIUM or SELF-SUPPORT
- **status_details**: Status detail string
- **subscription_type**: Subscription - type
- **starts**: Earliest date and time the subscription is valid for
- **ends**: When does this subscription expire
- **system_type**: Is this a phyiscal, container or virtual system?

### rhsm_pool Example

```puppet
rhsm_pool { '1a2b3c4d5e6f1234567890abcdef12345':
  name              => 'Extra Packages for Enterprise Linux',
  ensure            => present,
  provides          => 'EPEL',
  sku               => 1234536789012,
  contract          => 'Fancy Widgets, LTD',
  account           => '1234-12-3456-0001',
  serial            => 1234567890123456789,
  id                => 1a2b3c4d5e6f1234567890abcdef12345,
  active            => true,
  quantity_used     => 1,
  service_level     => 'STANDARD',
  service_type      => 'EOL',
  status_details    => 'expired',
  subscription_type => 'permanent',
  starts            => 06/01/2015,
  ends              => 05/24/2045,
  system_type       => physical,
}
```

## Installing

In your puppet modules directory:

    git clone https://github.com/jlaska/puppet-subscription_manager.git

Ensure the module is present in your puppetmaster's own environment (it doesn't
have to use it) and that the master has pluginsync enabled.  Run the agent on
the puppetmaster to cause the custom types to be synced to its local libdir
(`puppet master --configprint libdir`) and then restart the puppetmaster so it
loads them.

## Issues

Please file any issues or suggestions on [on GitHub](https://github.com/jlaska/puppet-subscription_manager/issues)
