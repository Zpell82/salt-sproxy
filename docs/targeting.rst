.. _targeting:

Targeting
=========

Targeting devices is largely based on the Roster interface. This is the 
starting point, where *salt-sproxy* know what devices you want to manage. The 
Roster interface can be a Pillar file, an Ansible inventory file, a NetBox 
instance and so on. See :ref:`using-roster` for more details, usage examples 
and documentation for each of the available Roster options.

To put it in other words, the Roster provides the totality (or the universe) of
devices you have. When you're executing a command, you may want to execute 
a command against all these devices, or only a subset of them. There are 
several targeting selection mechanisms, as presented below.

Targeting in *salt-sproxy*, from an user perspective, is very similar to the 
native Salt targeting - however, the implementation is fundamentally different 
(again, please see :ref:`using-roster` for more details on this); that's why 
the targeting in *salt-sproxy* comes with some caveats you should be aware of.

.. tip::

    Before executing any command, it may be a good idea to check that your 
    target matches the devices you want to run against, by using the 
    ``--preview-target`` CLI option, e.g.,

    .. code-block:: bash

        salt-sproxy -G netbox:role:router --preview-target

.. _targeting-caveats:

.. seealso::

    When targeting making use of Grains or Pillar data that depend on the device 
    characteristics (such as interfaces, IP addresses, OS version, platform 
    details, and so on), or other properties retrieved from other systems, such 
    as APIs, databases, etc., *salt-sproxy* needs cached data which is obtained 
    _only_ after the (first) execution is complete. To do so, you have a number 
    of options: once *salt-sproxy* is installed and configured, you can run 
    ``salt-sproxy '*' test.ping`` or similar to execute across all the devices 
    and cache their data.

    Alternatively, you may want to look at
    `--invasive-targeting 
    <https://salt-sproxy.readthedocs.io/en/latest/opts.html#cmdoption-invasive-targeting>`__
    or `--preload-targeting 
    <https://salt-sproxy.readthedocs.io/en/latest/opts.html#cmdoption-preload-targeting>`__
    options. These two options would allow you to work around this caveat, 
    however bear in mind they'll eveluate - and eventually connect to - every 
    device *salt-sproxy* is aware of, in order to determine which devices match 
    your target.

.. _targeting-glob:

Glob
----

Shell-style globbing on the device name / Minion ID.

See https://docs.saltstack.com/en/latest/topics/targeting/globbing.html#globbing

Examples:

- Match all the devices *salt-sproxy* knows about:

.. code-block:: bash

    salt-sproxy '*' test.ping

- Match ``edge1`` and ``edge3`` devices:

.. code-block:: bash

    salt-sproxy 'edge[1,3]' test.ping

.. _targeting-pcre:

PCRE
----

PCRE stands for Perl Compatible Regular Expression, so you can target against 
devices with the name matching the regular expression.

See also: https://docs.saltstack.com/en/latest/topics/targeting/globbing.html#regular-expressions

Example: match top of rack switches with the name ending in a digit:

.. code-block:: bash

    salt-sproxy -E '.*-tor\d' napalm.junos_rpc get-route-summary-information table=mpls.0

.. _targeting-list:

List
----

A list of device names.

Example: execute a command on three devices ``edge1``, ``edge2``, and 
``edge3``:

.. code-block:: bash

    salt-sproxy -L 'edge1,edge2,edge3' net.arp

.. _targeting-grain:

Grain
-----

Targeting using Grain data.

This is a tricky subject. Unlike the native Salt, *salt-sproxy* doesn't have 
access to device data before connecting to it (i.e., it can't possibly know 
device details before even connecting to it). You can however target using 
Grain data, but there are some caveats, and it's up to you to decide whether 
you want performance or limit the resource consumption. Generally, Grain 
targeting won't work at the first execution, as *salt-sproxy* needs cached 
data. An alternative would be using the ``--invasive-targeting`` or 
``--preload-targeting`` options, but that has a price (see 
https://salt-sproxy.readthedocs.io/en/latest/opts.html#cmdoption-invasive-targeting 
for more details).

.. seealso::

    See also: :ref:`static-grains`. Static Grains are always available, and can 
    be anytime used in your targeting, without any restrictions.

An exception is the :ref:`netbox-roster` module which provides an additional 
set of Grains you can use, under the ``netbox`` key. See the
:ref:`netbox-roster-grain` section for more details.

Examples: match devices on their role:

.. code-block:: bash

    salt-sproxy -G role:router test.ping

.. _targeting-grain-pcre:

Grain PCRE
----------

As the ``grain`` targeting, but instead of exact matching, can match on 
a regular expression on the Grain value.

.. note::

    This targeting mechanism has the same caveats as the
    :ref:`targeting-grain`.

Example: match the devices from multiple sites (e.g., ``lon1``, ``lon2``, etc.)

.. code-block:: bash

    salt-sproxy -P site:lon\d test.ping

.. _targeting-pillar:

Pillar
------

Targeting using Pillar data.

Similarly to the Grain targeting, this is possible but with one caveats when
you're not running active Minions: *salt-sproxy* needs this data cached in 
order to evaluate the target and determine which devices match, however using 
the CLI (and configuration file) options ``--invasive-targeting`` or 
``--preload-targeting``, you can work around this limitation. Once you've 
executed once, the data will be cached, and you can use it for future targets.

.. hint::

    If you want to target against statically defined Pillar, whenever possible, 
    static Grains may be a better fit for your use case. Have a look at 
    :ref:`static-grains`.

Example:

.. code-block:: bash

    salt-sproxy -I proxy:user:salt --preview-target

.. _targeting-pillar-pcre:

Pillar PCRE
-----------

As the ``pillar`` targeting, but instead of exact matching, can match on 
a regular expression on the Pillar value.

.. note::

    This targeting mechanism has the same caveats as the
    :ref:`targeting-pillar`.

.. hint::

    If you want to target against statically defined Pillar, whenever possible, 
    static Grains may be a better fit for your use case. Have a look at 
    :ref:`static-grains`.

Example: match the devices from multiple sites, based on the hostname pattern 
(e.g., ``lon1``, ``lon2``, etc.)

.. code-block:: bash

    salt-sproxy -J proxy:host:.*lon\d --preview-target

.. _targeting-compound:

Compound
--------

You can mix all the matchers above. See 
https://docs.saltstack.com/en/latest/topics/targeting/compound.html for more 
details and notes.

Example: match edge routers 1 and 3 from multiple sites

.. code-block:: bash

    salt-sproxy -C 'edge[1,3] and G@role:router and P@site:lon\d' net.lldp
