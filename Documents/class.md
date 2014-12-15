﻿# Class 

[Language: Classes](https://docs.puppetlabs.com/puppet/latest/reference/lang_classes.html)

[Learning Puppet — Modules and Classes](https://docs.puppetlabs.com/learning/modules1.html#classes)

### Inheritance

https://docs.puppetlabs.com/puppet/latest/reference/lang_classes.html#inheritance

Class inheritance should be avoided. It should be used mainly to avoid repetition of code.

Cons:
- Decreases modularity
- Increases compile time

Pros:
- Allows to extend classes and use the local scope variables

When to use:
- Override resource attributes
- Avoid repetition of code

```puppet
# Apache file
class apache {
    service {'apache':
        require => Package['httpd'],
    }
}

# Apache SSL file
class apache::ssl inherits apache {
    # host certificate is required for SSL to function
    Service['apache'] {
        require +> [ File['apache.pem'], File['httpd.conf'] ],
        # Since `require` will retain its previous values, this is equivalent to:
        # require => [ Package['httpd'], File['apache.pem'], File['httpd.conf'] ],
    }
}
```

**Example 1a:** Calling a local scope variable from another class

```puppet
# manifests/params.pp 
class base::params {
	case $::osfamily {
        'RedHat': { $ssh_name = 'sshd' }
        'Debian': { $ssh_name = 'ssh' }
        default: { warning('OS not supported by puppet modle SSH') }
    }
}


# manifests/ssh.pp
class base::ssh {
    package { 'SSH':
        name    => 'openssh-server',
        ensure  => present,
    }

    file { 'sshd-config':
        name    => '/etc/ssh/sshd_config',
        ensure  => file,
        owner   => 'root',
        group   => 'root',
        source  => 'puppet:///modules/base/sshd_config',
        require => Package['SSH'],
        notify  => Service['ssh-service-name-two'],
    }

    service { 'sshd':
        name    => $base::params::ssh_name,
        ensure  => running,
        alias   => 'ssh-service-name-two',
        enable  => true,
    }
}
```

The main `site.pp` must include the class being called.

```puppet
# ../../manifests/site.pp 
node "xyz" {
    include base::params
    include base::ssh
}
```

**Example 1b:** Inheriting a class

```puppet
# manifests/params.pp 
class base::params {
    case $::osfamily {
        'RedHat': { $ssh_name = 'sshd' }
        'Debian': { $ssh_name = 'ssh' }
        default: { warning('OS not supported by puppet modle SSH') }
    }
}


# manifests/ssh.pp
class base::ssh inherits base::params {
    package { 'SSH':
        name    => 'openssh-server',
        ensure  => present,
    }

    file { 'sshd-config':
        name    => '/etc/ssh/sshd_config',
        ensure  => file,
        owner   => 'root',
        group   => 'root',
        source  => 'puppet:///modules/base/sshd_config',
        require => Package['SSH'],
        notify  => Service['ssh-service-name-two'],
    }

    service { 'sshd':
        name    => $base::params::ssh_name,
        ensure  => running,
        alias   => 'ssh-service-name-two',
        enable  => true,
    }
}
```

No need for `include base::params` in the main `site.pp`.

```puppet
# ../../manifests/site.pp 
node "xyz" {
    include base::ssh
}
```