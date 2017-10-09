

We would propose leveraging a proven, open, auditable Cloud eventing standard exists that considers many industry use cases and scenarios that have been developed by many companies jointly at the DMTF.

The CADF format is suitable for all auditable resource activities (e.g., CRUD+ APIs), IBM Cloud has moved to using an international standard Cloud Auditing Data Federation (CADF) event format as standardized at the Digital Mgmt. Task Force (DMTF). See https://www.dmtf.org/standards/cadf. It is fully compliant with all Security auditing standards, including ISO27000 series, PCI, etc.

It also considers many common event needs that otherwise would have to be re-invented , including:
  - federation / merge of events/logs across Clouds
  - data extensibility (via attachements, extended JSON fields, profiles, tagging).
  - event tagging (including geolocation)
  - metric events
  - extensible Cloud-neutral taxonimies for resource, action and outcomes
  - std. model for event observation/reporting
  - UUID for federated resource ID
  - ISO timestamps
  - event signing

Its primary format is JSON; here is a sample pseudo-event:

{
  "typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event", 
  "id": "myscheme://mydomain/event/id/1234", 
  "eventType": "activity", 
  "eventTime": "2012-03-22T13:00:00-04:00",   // ISO standard
  "action": "create", 
  "outcome": "success", 
  "initiator": {
    "id": "myuuid://location.org/resource/0001",
    "typeURI": "..."   // extensible tax. value
  },
  "target": {
    "id": "myuuid://location.org/resource/0099",
    "typeURI": "..."   // extensible tax. value
    },
  "observer": {
    "id": "myuuid://location.org/resource/0321",
    "typeURI": "..."   // extensible tax. value
    },
  "reporterchain": [  // optional
  { 
    "role": "observer", 
    "reporterTime": "2012-08-22T23:00:00-02:00", 
    "reporterId”: "..."
  },
  ...
  ]
}

Pleate note many fields are optional; when I have more time I can copy the grammar and table of fields.

more detailed CADF event examples forthcoming (which will include examles specific to IBM Cloud and its payload/field extensions).
