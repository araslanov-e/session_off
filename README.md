SessionOff
==========

If You're old enough in Rails age, You should remember the times when sessions
where not (that) lazy loaded and one could have turned them off declaratively.

I really liked this explicit approach and sure started missing it ever since it
got redesigned. Now in theory it sounds great that if You do not touch the session
it won't be loaded, but in practice enforcing Your code to not invoke the session 
method (and thus not send a session cookie) for a (session-less) mobile client 
while sharing the same code for "normal" (session-aware) clients, is just *hard*.
You need to come up with a session state management and even so it's non-trivial 
to track down "bugs" where You access the session while not really wanting to !

How about we bring the love back and allow (once again) for a session to be off :

    session :off                     # turn session management off
    session :off, :only => :bar      # only for :bar action
    session :off, :except => :bar    # except for :bar action
    session :only => :foo,           # on for :foo when doing HTTPS
            :session_secure => true 
    session :off, :only => :foo,     # off for foo, if uses as web service
            :if => Proc.new { |req| req.parameters[:ws] }

**NOTE:** Keep in mind that after installing the plugin `session` might be nil !
But only while using the `session` (instance) method in controllers, it does
leave the `request.session` as is. Accessing the `request.session` directly will
work (and load the session) no matter if the session is off. This is intentional 
as the goal of the plugin is to provide session management for controller code 
and not the whole middleware stack !
I also highly discourage against using `session` inside model classes, as it is a 
lack of good desing, after all session's just an abstracted request "extension".

If in a need to check for a "disabled" session state from a request use :

    request.session_options[:disabled]

If unhappy with what happens while the session is turned off for an action :

    class ApplicationController < ActionController::Base

      def disable_session
        super # sets @_session = false (disabled)
        send_mama_a_kiss # or whatever You want !
      end

    end


Install
=======

Installable as a plain-old rails plugin:

    script/plugin install git://github.com/kares/session_off.git

The plugin has been tested and should work on Rails **3.0** as well as **2.3**.

Example
=======

or how to be nice to robots :

    class RobotsController < ApplicationController

      # http://gurge.com/blog/2007/01/08/turn-off-rails-sessions-for-robots/
      ROBOTS = /\b(Baidu|Gigabot|Googlebot|libwww-perl|lwp-trivial|msnbot|SiteUptime|Slurp|WordPress|ZIBB|ZyBorg)\b/i

      # turn off sessions if this is a request from a robot
      session :off, :if => proc { |request| request.user_agent =~ ROBOTS }

    end

<http://log.kares.org/search/label/session_off>
