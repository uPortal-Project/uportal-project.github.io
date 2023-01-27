# Person Attribute Group Store (PAGS) Overview

## PAGS Overview

The PAGS defines group memberships by logical expressions on attributes retrieved by PersonDirectory. (PersonDirectory initializes IPersons with attributes retrieved from one or more sources of directory information, including, for example LDAP.)

## Capabilities of PAGS

PAGS computes entity memberships by testing the value of selected `IPerson` attributes. Like PersonDirectory, the PAGS retrieves information one user at a time. As a result, it can answer inquiries about what groups a particular `IPerson` or _group member_ belongs to, but it cannot answer inquiries about what entity members are contained by a given group.

### Can do:

  - contains()
  - find()
  - findContainingGroups()

### Can't do

- `findEntitiesForGroup()`

Although PAGS groups cannot answer their entity members, they are aware of their member groups.  So they can also do:

    - findMemberGroupKeys()
    - findMemberGroups()

This will usually suffice for authorization, so the PAGS can be thought of as an authorization-oriented group store. Since PersonDirectory supplies information about IPersons and not about ChannelDefinitions or other portal entities, a further limitation is that the PAGS can only contain memberships associating IPerson group members with IPerson groups.

The configuration lets you do 2 things: (i) declare IEntityGroups with key, name, description and member group keys, and (ii) declare the tests that determine if an IPerson is a member of a group. Since a group need not contain any member groups, the <members> element is optional. Since a group need not contain any member entities, the <selector-test> element is also optional.

Tests. Each of the tests to be applied to an IPerson is described in a <test> element, which consists of 3 required sub-elements, the name of the test class, the name of the attribute to be tested, and the value against which the attribute is to be tested by the tester class. In the example below, the test is true if the String value of the attribute "sn" equals "Jones".


```xml
 <test>
  <attribute-name>sn</attribute-name>
  <tester-class>org.jasig.portal.groups.pags.testers.StringEqualsTester</tester-class>
  <test-value>Jones</test-value>
 </test>
```

The PAGS currently ships with 8 tester classes, and you can easily create your own.  Each tester class must implement the IPersonTester interface, which consists of a single method:


```java
 public interface IPersonTester {
   public boolean test(IPerson person);
 }

```

The following tester classes (all in the package org.jasig.portal.groups.pags.testers) come with PAGS:

| Tester | Use |
| --- | ----------- |
| `AbstractIntegerTester` | Abstract base class for testers that test the values of an `IPerson` Integer attribute. |
| `AbstractNotValuesTester` | Base class for testers that determine membership based on the number of values a user has for an attribute. |
| `AbstractStringTester` | Abstract class tests a possibly multi-valued attribute against a test value. |
| `AdhocGroupTester` | Immutable PAGS Tester for inclusive/exclusive membership in sets of groups |
| `AlwaysTrueTester` |  |
| `BaseAttributeTester` | A tester for examining `IPerson` attributes. |
| `EagerRegexTester` | A tester for matching multiple values of an attribute against a regular expression. |
| `GuestUserTester` | Evaluates whether the user is a guest. |
| `InjectAttributeRegexTester` | A tester for matching the possibly multiple values of an attribute against a regular expression, and in replacing a pattern by an other user attribute (optional use). |
| `IntegerEQTester` | Converts attribute and test value to ints. Attribute must be EQ to the test value.  In the event of a `NumberFormatException`, the test fails (true for all Integer testers.) |
| `IntegerGETester` | Attribute must be GE test value. |
| `IntegerGTTester` | Attribute must be GT test value. |
| `IntegerLETester` | Attribute must be LE test value. |
| `IntegerLTTester` | Attribute must be LT test value. |
| `InvertedRegexTester` | A tester for matching the possibly multiple values of an attribute against a regular expression and returning true for a FAILURE to match. |
| `LowercasedRegexTester` | A tester for matching the possibly multiple values of an attribute (made lower case) against a regular expression. |
| `MissingAttributeTester` | Testing a missing attribute on `IPerson`. |
| `NbValuesEQTester` | A tester for examining if an `IPerson` attribute has exactly nth values. |
| `NbValuesGETester` | A tester for examining if an `IPerson` attribute has more or exactly nth values. |
| `NbValuesGTTester` | A tester for examining if an `IPerson` attribute has more than nth values. |
| `NbValuesLETester` | A tester for examining if an `IPerson` attribute has less or exactly nth values. |
| `NbValuesLTTester` | A tester for examining if an `IPerson` attribute has less than nth values. |
| `PropertyInvertedRegexTester` | A tester for matching the possibly multiple values of an attribute against a regular expression obtained from a property value from portal.properties and returning true for a FAILURE to match. |
| `PropertyRegexTester` | A tester for matching the possibly multiple values of an attribute against a regular expression obtained from a property value specified in portal.properties. |
| `RegexTester` | Attribute must match a regular expression.  Do not include the delimiter. |
| `StringEqualsIgnoreCaseTester` | String comparison ignoring case. |
| `StringEqualsTester` | String comparison. |
| `ThemeNameEqualsIgnoreCaseTester` | |
| `ValueExistsTester` | True if the attribute has any non-blank value. |
| `ValueMissingTester` | True if the attribute is null or none of its values equals the specified value. |

**Test Groups** Individual tests are aggregated into test groups, and their results are AND-ed together, so all tests in a test group must return true for the test group to return true. A test group is described by a <test-group> element. If there is more than 1 <test-group> element, the results of the test groups are OR-ed together, so if any one test group returns true, the tests return true.  A true result means that the candidate IPerson is a member of the group.

**Multi-valued attributes** The testers that come with PAGS will OR the tests of successive values of a multi-valued attribute, so if any value satisfies the test, the test returns true.  You can override this behavior in a custom tester.

**Caching of group store information (XML-based PAGS)**  Because the groups structure defined in the PAGS is invariant, an instance of each group, including the keys of its containing groups, is cached by the store on start-up.  These cached instances are dealt out to satisfy all requests to the store for groups.  As a result, any change to the store configuration (via _PAGSGroupStoreConfig.xml_) requires restarting the group service, which generally means restarting the portal.  By contrast, memberships for an IPerson group member are cached for the life of the corresponding user's portal session.  As a result, if an IPerson's attributes change, these new attributes -- and any group memberships they imply -- will be visible on next log in.

**uPortal 4.1 introduced an optional Entity-based PAGS**, which obtains PAGS data from the database rather than from an XML file.  When using Entity-based PAGS the aspects of the above caching note about requiring a redeploy/restart are not applicable to Entity-based PAGS.  However changes in an IPerson's attributes and any PAGS group membership accordingly still will not take effect until the user's next login.

**Recursive Testing**  If one PAGS group (the child group) belongs to another PAGS group (the parent group), then an IPerson member of the child group must also pass the test(s) required for membership in the parent group.  This is a more restrictive contract than the group system in general requires, where membership in a group only means that some relationship exists between a group and its members.  Here, when an entity group member belongs to a group, it means that the underlying entity has some specific attribute value(s).  The PAGS enforces this by testing the entity for membership in all its parent groups, which means you can nest the testing of entity members.  For example, assume a group named named "employees" requires status equal to "employed" and a group named "seniors" requires age GE 65.  If "seniors" is a member of "employees", then any IPerson member of "seniors" is required to have both age GE 65 and status equal to "employed".
