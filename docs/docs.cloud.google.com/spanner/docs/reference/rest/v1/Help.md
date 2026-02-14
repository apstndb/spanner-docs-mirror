  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [Link](#Link)
      - [JSON representation](#Link.SCHEMA_REPRESENTATION)

Provides links to documentation or for performing an out of band action.

For example, if a quota check failed with an error indicating the calling project hasn't enabled the accessed service, this can contain a URL pointing directly to the right place in the developer console to flip the bit.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;links&quot;: [
    {
      object (Link)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  links[]  `

`  object ( Link  ` )

URL(s) pointing to additional information on handling the current error.

## Link

Describes a URL link.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;description&quot;: string,
  &quot;url&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  description  `

`  string  `

Describes what the link offers.

`  url  `

`  string  `

The URL of the link.
