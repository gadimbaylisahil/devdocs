# Streaming downloads when using Apartment gem for multitenancy

Be careful when using streaming downloads while having [Apartment gem](https://github.com/rails-on-services/apartment) switching tenants. This will not work if it spawns a new thread, because, the new thread's connection will default to use the `.public` schema's data.

You will have to manually manage this and make sure the the code block that is streaming the data back to client is executed within `Apartment.switch(tenant) {}` block with correct tenant.