# Errors

The Proband Connect API may return the following errors:

Error Code | Meaning
---------- | -------
400 | Bad Request -- Generic bad request error
401 | Unauthorized -- Device or user is not authorized
403 | Forbidden -- User does not have permission to access resource
404 | Not Found -- The resource does not exist
406 | Not Acceptable -- You requested a format that isn't json
409 | Confict -- Attempting to create a resource that has already been created
410 | Resource Gone -- Resource has been deleted
412 | Unknown or already used UUID
500 | Internal Server Error -- Please report this to the Proband team

