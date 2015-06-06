# NAME

AnyEvent::SlackRTM - AnyEvent module for interacting with the Slack RTM API

# VERSION

version 0.151570

# SYNOPSIS

    use AnyEvent;
    use AnyEvent::SlackRTM;
    
    my $cond = AnyEvent->condvar;
    my $rtm = AnyEvent::SlackRTM->new($access_token);

    my $c = 1;
    my $keep_alive;
    my $counter;
    $rtm->on('hello' => sub { 
        print "Ready\n";

        $keep_alive = AnyEvent->timer(interval => 60, cb => sub {
            $rtm->ping;
        });

        $counter = AnyEvent->timer(interval => 5, cb => sub {
            $rtm->send({
                type => 'message',
                text => $i++, 
            });
        });
    });
    $rtm->on('message' => sub { 
        my ($rtm, $message) = @_;
        if ($message->{ok}) {
            print "> ", $message->{text};
        }
    });
    $rtm->on('finish' => sub { 
        print "Done\n";
        $cond->send;
    });

    $rtm->start;
    AnyEvent->condvar->recv;

# DESCRIPTION

This provides an [AnyEvent](https://metacpan.org/pod/AnyEvent)-based interface to the [Slack Real-Time Messaging API](https://api.slack.com/rtm). This allows a program to interactively send and receive messages of a WebSocket connection and takes care of a few of the tedious details of encoding and decoding messages.

As of this writing, the library is still a fairly low-level experience, but more pieces may be automated or simplified in the future.

**Somewhat Experimental:** The API here is not set in stone yet, so watch for surprises if you upgrade.

**Disclaimer:** Note also that this API is subject to rate limits and any service limitations and fees associated with your Slack service. Please make sure you understand those limitations before using this library.

# METHODS

## new

    method new($token)

Constructs a [AnyEvent::SlackRTM](https://metacpan.org/pod/AnyEvent::SlackRTM) object and returns it. 

The `$token` option is the access token from Slack to use. This may be either of the following type of tokens:

- [User Token](https://api.slack.com/tokens). This is a token to perform actions on behalf of a user account.
- [Bot Token](https://slack.com/services/new/bot). If you configure a bot integration, you may use the access token on the bot configuration page to use this library to act on behalf of the bot account. Bot accounts may not have the same features as a user account, so please be sure to read the Slack documentation to understand any differences or limitations.

## start

    method start()

This will establish the WebSocket connection to the Slack RTM service.

You should have registered any events using ["on"](#on) before doing this or you may miss some events that arrive immediately.

## metadata

    method metadata() returns HashRef

The initial connection is established after calling the [rtm.start](https://api.slack.com/methods/rtm.start) method on the web API. This returns some useful information, which is available here.

This will only contain useful information _after_ ["start"](#start) is called.

## quiet

    method quiet($quiet?) returns Bool

Normally, errors are sent to standard error. If this flag is set, that does not happen. It is recommended that you provide an error handler if you set the quiet flag.

## on

    method on($type, \&cb)

This sets up a callback handler for the named message type. The available message types are available in the [Slack Events](https://api.slack.com/events) documentation. Only one handler may be setup for each event. Setting a new handler with this method will replace any previously set handler. Events with no handler will be ignored/unhandled.

## off

    method off($type)

This removes the handler for the named `$type`.

## send

    method send(\%msg)

This sends the given message over the RTM socket. Slack requires that every message sent over this socket must have a unique ID set in the "id" key. You, however, do not need to worry about this as the ID will be set for you.

## ping

    method ping(\%msg)

This sends a ping message over the Slack RTM socket. You may add any paramters you like to `%msg` and the return "pong" message will echo back those parameters.

## said\_hello

    method said_hello() returns Bool

Returns true after the "hello" message has been received from the server.

## finished

    method finished() returns Bool

Returns true after the "finish" message has been received from the server (meaning the connection has been closed). If this is true, this object should be discarded.

## close

    method close()

This closes the WebSocket connection to the Slack RTM API.

# AUTHOR

Andrew Sterling Hanenkamp <hanenkamp@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2015 by Qubling Software LLC.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
