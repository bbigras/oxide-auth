Details breaking changes and migration possibilities. Report missing
information with an issue. For guides on larger migrations you may also request
more detailed information.

## v0.4.0-preview.1 [WIP]

Currently trying to streamline traits and method by making them less specialized
and reusable or hiding complexity behind a more succinct interface. Part of this
is cleaning up interfaces that were misguided by envisioning them with a too
heavy focus on optimization but sacrificing usability in the process. And as it
later turned out, this choice also introduced complicated inter-module
dependencies that reduced the overall available design space. Sacrificing
functionality for a small performance boost is not an acceptable tradeoff.

This migration notice denotes WIP or planned changes on the git version and will
be merged into a single log when a release is made. WIP changes may appear in
`preview.2` or later and are intended to provide advance notice of expected
interface changes.

-----

`Registrar` now requires the result of `bound_redirect` to have a lifetime
independent of `self`. Instead of requiring a long-living reference to an
internal `EncodedClient`, scope resolution has been moved to a separate method.

Rationale: This method required taking a borrow on whatever provided the
registrar due to its entangled lifetimes. Since the return value was needed
later in the authorization process the non-local borrow hindered simultaneous
usage of other mutable members, even if those were local.

At the same time, this change implies that the underlying storage of a
`Registrar` is no longer restricted to `EncodedClient` and can use arbitrary
data structures.

-----

The trait method `redirect_error` has been moved from `WebResponse` to
`Endpoint`. Its default implementation remains unchanged, simply converting into
a response and the error into the endpoint error.

Rationale: It is the endpoint that contains the implementation specific logic.
Since the error can only be enriched with additional information or references
to other pages before it is converted to a `WebResponse`, this logic is not
universal for implementations for `WebResponse` but rather needs customization
from the endpoint.

-----

The previous `OwnerAuthorizer` and `OwnerAuthorization` have been renamed to
`OwnerSolicitor` and `OwnerConsent` to avoid confusion arising from the use of
'authorization'. Additionally, it has been integrated in the endpoint next to
other primitives, no frontend actually supported very different usage anyways.

-----

[WIP] The `code_grant::frontend` flow design has been revamped into a unified
trait. This replaces the explicit `…Flow` constructors while allowing greater
customization of errors, especially allowing the frontend to react in a custom
manner to `primitive` errors or augment errors of its own type. This should open
the path to more flexible `future` based implementations.

-----

[WIP] QueryParamter is now a private struct with several `impl From<_>` as
substitutes for the previous enum variants. This should make frontend
implementation more straightforward by simply invoking `.into()` while allowing
introduction of additional representations in a non-breaking change (variants of
public enums are strictly speaking breaking changes while new impls of crate
types are not).

-----

The possibility to create a `TokenSigner` instance based on a password has been
removed. The use of this was discouraged all along but this removes another
possible security pit fall. Note that you may want to migrate to a self-created
`ring::hmac::SigningKey` instance, using a randomly generated salt and a
self-chosen, secure password based key derivation function to derive the key
bytes. Alternatively, you can now create a `TokenSigner` that uses an ephemeral
key, i.e. the key will change for each program invocation, invalidating all
issued tokens of other program runs.

Rationale: Password based, low-entropy keys do not provide adequate security.
While the interface offered the ability to provide a high-entropy salt to
create a secure signing key, it was easy not to do so (and done in the examples
against the recommendation of the documentation). The scenario for the
examples, and by extension maybe the scenario of users, did not rely on
persistent keys. The new interface should offer high security both for a
configuration-free setup and a production environment. Relying on the standard
constructors for `SigningKey` is intended to urge users to correctly use
high-entropy inputs such as the default rng of standard password hashing
algorithms.

## v0.4.0-preview0

Actix is the only fully supported framework for the moment as development begins
on trying to support `async` based frameworks. This release is highly
experimental and some breaking changes may go unnoticed due to fully switching
the set of frontends. Please understand and open a bug report if you have any
migration issues.