# NAME

Promise::ES6 - ES6-style promises in Perl

# SYNOPSIS

    $Promise::ES6::DETECT_MEMORY_LEAKS = 1;

    my $promise = Promise::ES6->new( sub {
        my ($resolve_cr, $reject_cr) = @_;

        # ..
    } );

    my $promise2 = $promise->then( sub { .. }, sub { .. } );

    my $promise3 = $promise->catch( sub { .. } );

    my $promise4 = $promise->finally( sub { .. } );

    my $resolved = Promise::ES6->resolve(5);
    my $rejected = Promise::ES6->reject('nono');

    my $all_promise = Promise::ES6->all( \@promises );

    my $race_promise = Promise::ES6->race( \@promises );

# DESCRIPTION

This is a rewrite of [Promise::Tiny](https://metacpan.org/pod/Promise::Tiny) that implements fixes for
certain bugs that proved hard to fix in the original code. This module
also removes superfluous dependencies on [AnyEvent](https://metacpan.org/pod/AnyEvent) and [Scalar::Util](https://metacpan.org/pod/Scalar::Util).

The interface is the same, except:

- Promise resolutions and rejections accept exactly one argument,
not a list. (This accords with the standard.)
- A `finally()` method is defined.
- Unhandled rejections are reported via `warn()`. (See below
for details.)

# COMPATIBILITY

Right now this doesn’t try for interoperability with other promise
classes. If that’s something you want, make a feature request.

# UNHANDLED REJECTIONS

As of version 0.05, unhandled rejections prompt a warning _only_ if one
of the following is true:

- 1) The unhandled rejection happens outside of the constructor.
- 2) The unhandled rejection happens via an uncaught exception
(even within the constructor).

# MEMORY LEAKS

It’s easy to create inadvertent memory leaks using promises in Perl.
Here are a few “pointers” (heh) to bear in mind:

- As of version 0.07, any Promise::ES6 instances that are created while
`$Promise::ES6::DETECT_MEMORY_LEAKS` is set to a truthy value are
“leak-detect-enabled”, which means that if they survive until their original
process’s global destruction, a warning is triggered.
- If your application needs recursive promises (e.g., to poll
iteratively for completion of a task), the `current_sub` feature (i.e.,
`__SUB__`) may help you avoid memory leaks.
- Garbage collection before Perl 5.18 seems to have been buggy.
If you work with such versions and end up chasing leaks,
try manually deleting as many references/closures as possible. See
`t/race_success.t` for a notated example.

    You may also (counterintuitively, IMO) find that this:

        my ($resolve, $reject);

        my $promise = Promise::ES6->new( sub { ($resolve, $reject) = @_ } );

        # … etc.

    … works better than:

        my $promise = Promise::ES6->new( sub {
            my ($resolve, $reject) = @_;

            # … etc.
        } );

# SEE ALSO

If you’re not sure of what promises are, there are several good
introductions to the topic. You might start with
[this one](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises).

Promise::ES6 serves much the same role as [Future](https://metacpan.org/pod/Future) but exposes
a standard, cross-language API rather than a proprietary one.

CPAN contains a number of other modules that implement promises.
Promise::ES6’s distinguishing features are simplicity and lightness.
By design, it implements **just** the standard Promise API and doesn’t
assume you use, e.g., [AnyEvent](https://metacpan.org/pod/AnyEvent).
