# SmartLDAP Group Store

## Purpose

The SmartLdapGroupStore is designed to discover groups of users maintained within an LDAP directory server. It queries an LDAP data source for a collection of group objects and uses their attributes to re-organize them into a hierarchy. As users log in and use the portal, it uses Person Attributes to determine which groups each user belongs to. This behavior is similar to PAGS

Several other implementations of GaP also leverage information from LDAP - PAGS, LDAPGroupStore, JitLDAPGroupStore - and each of these is useful in its own way. The role of SmartLdapGroupStore is to host in the portal groups that are defined and maintained entirely in LDAP. You don't have to state explicitly the groups you want to pull in; SmartLdap will automatically fetch all the groups that match the filter expression.

## Prerequisite

An LDAP Context Bean in the Spring Context is required. This can be either the `defaultLdapContext` or your own bean.

## Configuration

SmartLdapGroupStore must coordinate both with your directory server and Person Attributes in the portal, so it's easy to make a mistake setting it up. The following sections describe the necessary steps for using SmartLdap in the portal.

### Add SmartLdapGroupStore to Composite Group Services

If you do not have `compositeGroupServices.xml` in your uPortal-start repo at `overlays/uPortal/main/resources/properties/groups/`
then you will need to pull in a copy from the uPortal repo `uPortal-webapp/src/main/resources/properties/groups/`.
(You can copy from GitHub without cloning the repo locally.)

Add (or un-comment) the following entry in uportal-war/src/main/resources/properties/groups/compositeGroupServices.xml:

```xml
  <service>
    <name>smartldap</name>
    <service_factory>org.apereo.portal.groups.ReferenceIndividualGroupServiceFactory</service_factory>
    <entity_store_factory>org.apereo.portal.groups.smartldap.SmartLdapEntityStore$Factory</entity_store_factory>
    <group_store_factory>org.apereo.portal.groups.smartldap.SmartLdapGroupStore$Factory</group_store_factory>
    <entity_searcher_factory>org.apereo.portal.groups.smartldap.SmartLdapEntitySearcher$Factory</entity_searcher_factory>
    <internally_managed>false</internally_managed>
    <caching_enabled>true</caching_enabled>
  </service>
```

### Configure SmartLdapGroupStore

To configure SmartLdapGroupStore implement a bean of class `org.apereo.portal.groups.smartldap.SmartLdapGroupStore`
with a name of `smartLdapGroupStore`. 

Here is a summary of the settings for this bean:

  - `ldapContext`: reference to the LDAP Context bean to use
  - `memberOfAttributeName`: Name of the Person Attribute on each user that contains the distinguishedName of each group he/she is a member of (e.g. 'memberOf')
  - `baseGroupDn`: DN fragment that will be prepended to the Base DN of the `ldapContext` under which to query for group records
  - `childGroupKeyRegex`: Regex to extract the ID of the group
  - `displayPersonMembers`: ??  (true|false)
  - `filter`: LDAP filter for what objects to evaluate(e.g. '(objectCategory=group)', '(objectClass=groupOfNames)')
  - `groupsTreeRefreshIntervalSeconds`: how frequently should SmartLdap check for changes to the groups hierarchy?
  - `resolveMemberGroups`: Whether you want to resolve member groups outside the original Base DN (true/false)
  - `resolveDn`: another Base DN for resolving member groups if `resolveMemberGroups` (above) it true
  - `attributeMapper` > `keyAttributeName`: LDAP attribute on each group that contains it's distinguished name
  - `attributeMapper` > `groupNameAttributeName`: LDAP attribute on each group that contains it's human-readable name
  - `attributeMapper` > `membershipAttributeName`: LDAP attribute on each group that contains distinguished name of each of it's members

Example `attributesMapper` Bean:

```xml
    <bean id="smartLdapGroupStore" class="org.apereo.portal.groups.smartldap.SmartLdapGroupStore" lazy-init="false"
          depends-on="defaultLdapContext">

        <!--
         | This property is the ContextSource instance that will be used to connect to LDAP.  Uncomment the
         | following to use the LDAP settings in ldapContext.xml for user attributes, or supply your own.
         +-->
        <!--
        -->
        <property name="ldapContext" ref="defaultLdapContext"/>

        <!--
         | This property identifies the name of the Person Attribute that
         | lists the SmartLdap groups each person is a member of.
         +-->
        <property name="memberOfAttributeName" value="memberOf"/>

        <!--
         | BaseDn that will be passed to the search (not to the context).
         |
         | WARNING:  If you get an error like this...
         |   ...PartialResultException: [LDAP: error code 10...
         | it probably means your baseGroupDn isn't correct!
         +-->
        <property name="baseGroupDn" value="ou=Groups"/>

        <!--
         | This parameter is used to extract the id path of the groups
         | from their dn. The id path is expected to be catched from the first group
         | of the regex.
        <property name="childGroupKeyRegex" value="cn=(.*),ou=Groups,dc=apereo,dc=org"/>
        <property name="childGroupKeyRegex" value="cn=(.*),ou=.*,dc=apereo,dc=org"/>
         +-->
        <property name="childGroupKeyRegex" value="cn=(.*),ou=Groups,dc=apereo,dc=org"/>

        <!--
         | This parameter is used to avoid performance problems with large people's members groups
         | on groups manager. Default value is false, so comment to disable.
         +-->
        <!--
        -->
        <property name="displayPersonMembers" value="true"/>
        <!--
         | Group Tree Separator is used to set the char that is used to compose
         | Group Id in grouper that represent each tree paths.
         | Override it if the default value ":" doesn't match
        -->
        <!--<property name="groupTreeSeparator" value=":" />-->

        <!--
         | LDAP query string that will be passed to the search.
         +-->
        <property name="filter" value="(objectClass=groupOfNames)"/>

        <!--
         | Period, in seconds, after which SmartLdap will drop and re-init the groups
         | tree.  A value of zero or less (negative) disables this feature.
         +-->
        <property name="groupsTreeRefreshIntervalSeconds" value="7200"/>

        <!--
         | These next 2 properties tell smartLdap whether to gather additional groups that
         | are members of groups returned by the first baseGroupDn and filter, and where to
         | look if so.
         |
         |   - resolveMemberGroups=[true|false]
         |   - resolveDn={a different, broader baseGroupDn than the one above}
         |
         | Here's how it works:  smartLdap will first collect all groups under the
         | baseGroupDn specified above.  If 'resolveMemberGroups' is enabled, it will
         | also search for additional groups (found within the 'resolveDn' specified
         | here) that are members of groups in the first collection.
         +-->
        <property name="resolveMemberGroups" value="false"/>
        <property name="resolveDn" value="ignored"/><!--Used with resolveMemberGroups -->

        <!--
         | This property identifies the org.springframework.ldap.core.AttributesMapper
         | implementation used in reading the groups records from LDAP.
         +-->
        <property name="attributesMapper">
            <bean id="attributesMapper" class="org.apereo.portal.groups.smartldap.SimpleAttributesMapper">
                <!--
                 | Name of the group attribute that tells you its key.
                 +-->
                <property name="keyAttributeName">
                    <value>distinguishedName</value>
                </property>
                <!--
                 | Name of the group attribute that tells you its name.
                 +-->
                <property name="groupNameAttributeName">
                    <value>cn</value>
                </property>
                <!--
                 | Name of the group attribute that lists its members.
                 +-->
                <property name="membershipAttributeName">
                    <value>member</value>
                </property>
            </bean>
        </property>
    </bean>
```

If your LDAP schema doesn't manage groups and their relationships to each other in this way, you can implement a custom org.springframework.ldap.core.AttributesMapper class that bridges the difference.

## Configure Person Attribute DAO(s)

One of your Person Attribute DAOs must define a user attribute that matches `memberOfAttributeName` configured above that contains the groups
which users are a member of.

