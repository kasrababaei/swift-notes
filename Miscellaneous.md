# Miscellaneous

## Download Test Files

In case we want to hit an endpoint to downlooad a large file to test, for instance, a progress view or task cancellation, [this website](http://xcal1.vodafone.co.uk) offers a few endpoints.

The only catch is that the endpoint isn't using SSL pinning; therefore, the project needs to add [NSAppTransportSecurity](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity/).
