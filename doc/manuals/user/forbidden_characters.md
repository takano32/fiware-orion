# Forbidden characters

## General restrictions

In order to avoid script injections attack in some circumstances (e.g.
cross domain to co-located web servers in the same hot that CB) the
following characters are forbidden in any request:

-   &lt;
-   &gt;
-   "
-   '
-   =
-   ;
-   (
-   )

Any attempt of using them will result in a 400 Bad Request response
like this:

    {
        "error": "BadRequest",
        "description": "Invalid characters in attribute type"
    }

If your application needs to use these characters, you should encode it
using a scheme not including forbidden characters before sending the
request to Orion. 

[URL encoding](http://www.degraeve.com/reference/urlencoding.php) is
a valid way of encoding. However, we don't recommend its usage for
fields that may appear in API URL (such as entity id or attribute names)
due to it would need to encode the "%" character itself. For instance,
if we would want to use "E<01>" as entity id, its URL encode would be:
"E%3C01%3E".

In order to use this entity ID in URL (e.g. a retrieve entity info operation)
the following will be used (note that "%25" is the encoding for "%").

```
GET /v2/entities/E%253C01%253E
```

### Exceptions

There are some exception cases in which the above restrictions do not apply. In particular, in the following fields:

* URL parameter `q` allows the special characters needed by the Simple Query Language
* URL parameter `mq` allows the special characters needed by the Simple Query Language
* URL parameter `georel` and `coords` allow `;`
* Whichever attribute value which uses "TextUnrestricted" as attribute type. It could have security
  implications, use it at your own risk!

## Specific restrictions for ID fields

NGSIv2 introduces syntax restrictions for ID fields (such as entity id/type, attribute name/type
or metadata name/type) which are described in the "Field syntax restrictions" section in the
[NGSIv2 specification](http://telefonicaid.github.io/fiware-orion/api/v2/stable).

## Custom payload and headers special treatment

NGSIv2 provides a templating mechanism for subscriptions which allows to generate custom notifications
(see "Custom notifications" section in
the [NGSIv2 specification](http://telefonicaid.github.io/fiware-orion/api/v2/stable)). Forbidden
characters restrictions apply to the `httpCustom.payload` field in NGSIv2 API operations, such as
POST /v2/subscription or GET /v2/subscriptions. The same restrictions apply to the header values
in `httpCustom.headers`.

However, at notification time, any URL encoded characters in `httpCustom.payload` or in the values
of `httpCustom.headers` are decoded.

Example:

Let's consider the following `notification.httpCustom` object in a given subscription.

```
"httpCustom": {
  "url": "http://foo.com/entity/${id}",
  "headers": {
    "Content-Type": "text/plain",
    "Authorization": "Basic ABC...ABC%3D%3D"
  },
  "method": "PUT",
  "qs": {
    "type": "${type}"
  },
  "payload": "the value of the %22temperature%22 attribute %28of type Number%29 is ${temperature}"
}
```

Note that the above payload value is the URL encoded version of this string:
`the value of the "temperature" attribute (of type Number) is ${temperature}`. Note also that
`"Basic ABC...ABC%3D%3D"` is the URL encoded version of this string: `"Basic ABC...ABC=="`.

Now, let's consider that NGSIv2 implementation triggers a notification associated to this subscription.
Notification data is for entity with id `DC_S1-D41` and type `Room`, including an attribute named
`temperature` with value 23.4. The resulting notification after applying the template would be:

```
PUT http://foo.com/entity/DC_S1-D41?type=Room
Authorization: "Basic ABC...ABC=="
Content-Type: application/json 
Content-Length: 65

the value of the "temperature" attribute (of type Number) is 23.4
```
