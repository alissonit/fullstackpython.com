title: flask.globals current_app code examples
category: page
slug: flask-globals-current-app-examples
sortorder: 500021006
toc: False
sidebartitle: flask.globals current_app
meta: Python example code for the current_app function from the flask.globals module of the Flask project.


current_app is a function within the flask.globals module of the Flask project.


## Example 1 from Flask AppBuilder
[Flask-AppBuilder](https://github.com/dpgaspar/Flask-AppBuilder)
([documentation](https://flask-appbuilder.readthedocs.io/en/latest/)
and
[example apps](https://github.com/dpgaspar/Flask-AppBuilder/tree/master/examples))
is a web application generator that uses Flask to automatically create
the code for database-driven applications based on parameters set
by the user. The generated applications include default security settings,
forms, and internationalization support.

Flask App Builder is provided under the
[BSD 3-Clause "New" or "Revised" license](https://github.com/dpgaspar/Flask-AppBuilder/blob/master/LICENSE).

[**Flask AppBuilder / flask_appbuilder / menu.py**](https://github.com/dpgaspar/Flask-AppBuilder/blob/master/flask_appbuilder/./menu.py)

```python
# menu.py
from typing import List

~~from flask import current_app, url_for
from flask_babel import gettext as __

from .api import BaseApi, expose
from .basemanager import BaseManager
from .security.decorators import permission_name, protect


class MenuItem(object):
    def __init__(self, name, href="", icon="", label="", childs=None, baseview=None):
        self.name = name
        self.href = href
        self.icon = icon
        self.label = label
        self.childs = childs or []
        self.baseview = baseview

    def get_url(self):
        if not self.href:
            if not self.baseview:
                return ""
            else:
                return url_for(f"{self.baseview.endpoint}.{self.baseview.default_view}")
        else:
            try:


## ... source file abbreviated to get to current_app examples ...


    @property
    def reverse(self):
        return "navbar-inverse" in self.extra_classes

    def get_list(self):
        return self.menu

    def get_flat_name_list(self, menu: "Menu" = None, result: List = None) -> List:
        menu = menu or self.menu
        result = result or []
        for item in menu:
            result.append(item.name)
            if item.childs:
                result.extend(self.get_flat_name_list(menu=item.childs, result=result))
        return result

    def get_data(self, menu=None):
        menu = menu or self.menu
        ret_list = []

~~        allowed_menus = current_app.appbuilder.sm.get_user_menu_access(
            self.get_flat_name_list()
        )

        for i, item in enumerate(menu):
            if item.name == "-" and not i == len(menu) - 1:
                ret_list.append("-")
            elif item.name not in allowed_menus:
                continue
            elif item.childs:
                ret_list.append(
                    {
                        "name": item.name,
                        "icon": item.icon,
                        "label": __(str(item.label)),
                        "childs": self.get_data(menu=item.childs),
                    }
                )
            else:
                ret_list.append(
                    {
                        "name": item.name,
                        "icon": item.icon,
                        "label": __(str(item.label)),
                        "url": item.get_url(),


## ... source file abbreviated to get to current_app examples ...


                )
                self.find(category).childs.append(new_menu_item)

    def add_separator(self, category=""):
        menu_item = self.find(category)
        if menu_item:
            menu_item.childs.append(MenuItem("-"))
        else:
            raise Exception(
                "Menu separator does not have correct category {}".format(category)
            )


class MenuApi(BaseApi):
    resource_name = "menu"

    @expose("/", methods=["GET"])
    @protect(allow_browser_login=True)
    @permission_name("get")
    def get_menu_data(self):
~~        return self.response(200, result=current_app.appbuilder.menu.get_data())


class MenuApiManager(BaseManager):
    def register_views(self):
        if self.appbuilder.app.config.get("FAB_ADD_MENU_API", True):
            self.appbuilder.add_api(MenuApi)



## ... source file continues with no further current_app examples...

```


## Example 2 from FlaskBB
[FlaskBB](https://github.com/flaskbb/flaskbb)
([project website](https://flaskbb.org/)) is a [Flask](/flask.html)-based
forum web application. The web app allows users to chat in an open
message board or send private messages in plain text or
[Markdown](/markdown.html).

FlaskBB is provided as open source
[under this license](https://github.com/flaskbb/flaskbb/blob/master/LICENSE).

[**FlaskBB / flaskbb / auth / views.py**](https://github.com/flaskbb/flaskbb/blob/master/flaskbb/auth/views.py)

```python
# views.py
import logging
from datetime import datetime

~~from flask import Blueprint, current_app, flash, g, redirect, request, url_for
from flask.views import MethodView
from flask_babelplus import gettext as _
from flask_login import (
    confirm_login,
    current_user,
    login_fresh,
    login_required,
    login_user,
    logout_user,
)

from flaskbb.auth.forms import (
    AccountActivationForm,
    ForgotPasswordForm,
    LoginForm,
    LoginRecaptchaForm,
    ReauthForm,
    RegisterForm,
    RequestActivationForm,
    ResetPasswordForm,
)
from flaskbb.extensions import db, limiter
from flaskbb.utils.helpers import (
    anonymous_required,


## ... source file abbreviated to get to current_app examples ...


                )
                confirm_login()
                flash(_("Reauthenticated."), "success")
                return redirect_or_next(current_user.url)
            except StopAuthentication as e:
                flash(e.reason, "danger")
            except Exception:
                flash(_("Unrecoverable error while handling reauthentication"))
                raise

        return render_template("auth/reauth.html", form=form)


class Register(MethodView):
    decorators = [anonymous_required, registration_enabled]

    def __init__(self, registration_service_factory):
        self.registration_service_factory = registration_service_factory

    def form(self):
~~        current_app.pluggy.hook.flaskbb_form_registration(form=RegisterForm)
        form = RegisterForm()

        form.language.choices = get_available_languages()
        form.language.default = flaskbb_config['DEFAULT_LANGUAGE']
        form.process(request.form)  # needed because a default is overriden
        return form

    def get(self):
        return render_template("auth/register.html", form=self.form())

    def post(self):
        form = self.form()
        if form.validate_on_submit():
            registration_info = UserRegistrationInfo(
                username=form.username.data,
                password=form.password.data,
                group=4,
                email=form.email.data,
                language=form.language.data
            )

            service = self.registration_service_factory()
            try:
                service.register(registration_info)
            except StopValidation as e:
                form.populate_errors(e.reasons)
                return render_template("auth/register.html", form=form)
            except PersistenceError:
                    logger.exception("Database error while persisting user")
                    flash(
                        _(
                            "Could not process registration due"
                            "to an unrecoverable error"
                        ), "danger"
                    )

                    return render_template("auth/register.html", form=form)

~~            current_app.pluggy.hook.flaskbb_event_user_registered(
                username=registration_info.username
            )
            return redirect_or_next(url_for('forum.index'))

        return render_template("auth/register.html", form=form)


class ForgotPassword(MethodView):
    decorators = [anonymous_required]
    form = ForgotPasswordForm

    def __init__(self, password_reset_service_factory):
        self.password_reset_service_factory = password_reset_service_factory

    def get(self):
        return render_template("auth/forgot_password.html", form=self.form())

    def post(self):
        form = self.form()
        if form.validate_on_submit():

            try:
                service = self.password_reset_service_factory()
                service.initiate_password_reset(form.email.data)


## ... source file continues with no further current_app examples...

```


## Example 3 from flask-base
[flask-base](https://github.com/hack4impact/flask-base)
([project documentation](http://hack4impact.github.io/flask-base/))
provides boilerplate code for new [Flask](/flask.html) web apps.
The purpose of the boilerplate is to stitch together disparate
libraries that are commonly used in Flask projects, such as
[Redis](/redis.html) for fast caching and transient data storage,
[SendGrid](https://www.twilio.com/sendgrid) for transactional email,
[SQLAlchemy](/sqlalchemy.html) for persistent data storage through a
[relational database](/databases.html) backend,
[Flask-WTF](https://flask-wtf.readthedocs.io/en/stable/) for form
handling and many others.

flask-base is provided as open source under the
[MIT license](https://github.com/hack4impact/flask-base/blob/master/LICENSE.md).

[**flask-base / app / models / user.py**](https://github.com/hack4impact/flask-base/blob/master/app/models/user.py)

```python
# user.py
~~from flask import current_app
from flask_login import AnonymousUserMixin, UserMixin
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from itsdangerous import BadSignature, SignatureExpired
from werkzeug.security import check_password_hash, generate_password_hash

from .. import db, login_manager


class Permission:
    GENERAL = 0x01
    ADMINISTER = 0xff


class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    index = db.Column(db.String(64))
    default = db.Column(db.Boolean, default=False, index=True)
    permissions = db.Column(db.Integer)
    users = db.relationship('User', backref='role', lazy='dynamic')

    @staticmethod
    def insert_roles():


## ... source file abbreviated to get to current_app examples ...


            db.session.add(role)
        db.session.commit()

    def __repr__(self):
        return '<Role \'%s\'>' % self.name


class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    confirmed = db.Column(db.Boolean, default=False)
    first_name = db.Column(db.String(64), index=True)
    last_name = db.Column(db.String(64), index=True)
    email = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))

    def __init__(self, **kwargs):
        super(User, self).__init__(**kwargs)
        if self.role is None:
~~            if self.email == current_app.config['ADMIN_EMAIL']:
                self.role = Role.query.filter_by(
                    permissions=Permission.ADMINISTER).first()
            if self.role is None:
                self.role = Role.query.filter_by(default=True).first()

    def full_name(self):
        return '%s %s' % (self.first_name, self.last_name)

    def can(self, permissions):
        return self.role is not None and \
            (self.role.permissions & permissions) == permissions

    def is_admin(self):
        return self.can(Permission.ADMINISTER)

    @property
    def password(self):
        raise AttributeError('`password` is not a readable attribute')

    @password.setter
    def password(self, password):
        self.password_hash = generate_password_hash(password)

    def verify_password(self, password):


## ... source file continues with no further current_app examples...

```


## Example 4 from flask-debugtoolbar
[Flask Debug-toolbar](https://github.com/flask-debugtoolbar/flask-debugtoolbar)
([documentation](https://flask-debugtoolbar.readthedocs.io/en/latest/)
and
[PyPI page](https://pypi.org/project/Flask-DebugToolbar/))
is a [Flask](/flask.html) conversion of the popular
[Django Debug Toolbar](https://github.com/jazzband/django-debug-toolbar)
project. This extension creates a sidebar with useful debugging
information when you are running a Flask application in development
mode. The project is provided as open source under
[this license](https://github.com/flask-debugtoolbar/flask-debugtoolbar/blob/master/LICENSE).

[**flask-debugtoolbar / flask_debugtoolbar / __init__.py**](https://github.com/flask-debugtoolbar/flask-debugtoolbar/blob/master/flask_debugtoolbar/./__init__.py)

```python
# __init__.py
import os
import warnings

~~from flask import Blueprint, current_app, request, g, send_from_directory, url_for
from flask.globals import _request_ctx_stack
from jinja2 import Environment, PackageLoader
from werkzeug.urls import url_quote_plus

from flask_debugtoolbar.compat import iteritems
from flask_debugtoolbar.toolbar import DebugToolbar
from flask_debugtoolbar.utils import decode_text

try:
    from importlib.metadata import version

    __version__ = version("Flask-DebugToolbar")
except ImportError:
    import pkg_resources

    __version__ = pkg_resources.get_distribution("Flask-DebugToolbar").version


module = Blueprint('debugtoolbar', __name__)


def replace_insensitive(string, target, replacement):
    no_case = string.lower()
    index = no_case.rfind(target.lower())


## ... source file abbreviated to get to current_app examples ...


        app = current_app

        if req.routing_exception is not None:
            app.raise_routing_exception(req)

        rule = req.url_rule

        if getattr(rule, 'provide_automatic_options', False) \
           and req.method == 'OPTIONS':
            return app.make_default_options_response()

        view_func = app.view_functions[rule.endpoint]
        view_func = self.process_view(app, view_func, req.view_args)

        return view_func(**req.view_args)

    def _show_toolbar(self):
        if request.blueprint == 'debugtoolbar':
            return False

~~        hosts = current_app.config['DEBUG_TB_HOSTS']
        if hosts and request.remote_addr not in hosts:
            return False

        return True

    def send_static_file(self, filename):
        return send_from_directory(self._static_dir, filename)

    def process_request(self):
        g.debug_toolbar = self

        if not self._show_toolbar():
            return

        real_request = request._get_current_object()

        self.debug_toolbars[real_request] = (
            DebugToolbar(real_request, self.jinja_env))

        for panel in self.debug_toolbars[real_request].panels:
            panel.process_request(real_request)

    def process_view(self, app, view_func, view_kwargs):
        real_request = request._get_current_object()
        try:
            toolbar = self.debug_toolbars[real_request]
        except KeyError:
            return view_func

        for panel in toolbar.panels:
            new_view = panel.process_view(real_request, view_func, view_kwargs)
            if new_view:
                view_func = new_view

        return view_func

    def process_response(self, response):
        real_request = request._get_current_object()
        if real_request not in self.debug_toolbars:
            return response

~~        if current_app.config['DEBUG_TB_INTERCEPT_REDIRECTS']:
            if response.status_code in self._redirect_codes:
                redirect_to = response.location
                redirect_code = response.status_code
                if redirect_to:
                    content = self.render('redirect.html', {
                        'redirect_to': redirect_to,
                        'redirect_code': redirect_code
                    })
                    response.content_length = len(content)
                    response.location = None
                    response.response = [content]
                    response.status_code = 200

        if not (response.status_code == 200 and
                response.is_sequence and
                response.headers['content-type'].startswith('text/html')):
            return response

        response_html = response.data.decode(response.charset)

        no_case = response_html.lower()
        body_end = no_case.rfind('</body>')

        if body_end >= 0:


## ... source file continues with no further current_app examples...

```


## Example 5 from flask_jsondash
[Flask JSONDash](https://github.com/christabor/flask_jsondash) is a
configurable web application built in Flask that creates charts and
dashboards from arbitrary API endpoints. Everything for the web app
is configured in JSON. The code is provided as open source under the
[MIT license](https://github.com/christabor/flask_jsondash/blob/master/LICENSE).

[**flask_jsondash / flask_jsondash / charts_builder.py**](https://github.com/christabor/flask_jsondash/blob/master/flask_jsondash/./charts_builder.py)

```python
# charts_builder.py


import json
import os
import uuid
from datetime import datetime as dt

import jinja2
~~from flask import (Blueprint, current_app, flash, redirect, render_template,
                   request, send_from_directory, url_for)

from flask_jsondash import static, templates

from flask_jsondash import db
from flask_jsondash import settings
from flask_jsondash.utils import setting
from flask_jsondash.utils import adapter
from flask_jsondash import utils
from flask_jsondash.schema import (
    validate_raw_json, InvalidSchemaError,
)

TEMPLATE_DIR = os.path.dirname(templates.__file__)
STATIC_DIR = os.path.dirname(static.__file__)

REQUIRED_STATIC_FAMILES = ['D3']

charts = Blueprint(
    'jsondash',
    __name__,
    template_folder=TEMPLATE_DIR,
    static_url_path=STATIC_DIR,
    static_folder=STATIC_DIR,
)


def auth(**kwargs):
~~    if 'JSONDASH' not in current_app.config:
        return True
~~    if 'auth' not in current_app.config['JSONDASH']:
        return True
    authtype = kwargs.pop('authtype')
~~    auth_conf = current_app.config.get('JSONDASH').get('auth')
    if authtype not in auth_conf:
        return True
    return auth_conf[authtype](**kwargs)


def metadata(key=None, exclude=[]):
    _metadata = dict()
~~    conf = current_app.config
    conf_metadata = conf.get('JSONDASH', {}).get('metadata')
    if key is not None:
        if key in conf_metadata:
            return conf_metadata[key]()
        else:
            return None
    for k, func in conf_metadata.items():
        if k in exclude:
            continue
        _metadata[k] = conf_metadata[k]()
    return _metadata


def local_static(chart_config, static_config):
    js_path = static_config.get('js_path')
    css_path = static_config.get('css_path')
    for family, config in chart_config.items():
        if config['js_url']:
            for i, url in enumerate(config['js_url']):
                url = '{}{}'.format(js_path, url.split('/')[-1])
                config['js_url'][i] = url_for('static', filename=url)
        if config['css_url']:
            for i, url in enumerate(config['css_url']):
                url = '{}{}'.format(css_path, url.split('/')[-1])


## ... source file continues with no further current_app examples...

```


## Example 6 from flask-login
[Flask-Login](https://github.com/maxcountryman/flask-login)
([project documentation](https://flask-login.readthedocs.io/en/latest/)
and [PyPI package](https://pypi.org/project/Flask-Login/))
is a [Flask](/flask.html) extension that provides user session
management, which handles common tasks such as logging in
and out of a [web application](/web-development.html) and
managing associated user session data. Flask-Login is
open sourced under the
[MIT license](https://github.com/maxcountryman/flask-login/blob/master/LICENSE).

[**flask-login / flask_login / utils.py**](https://github.com/maxcountryman/flask-login/blob/master/flask_login/./utils.py)

```python
# utils.py


import hmac
from hashlib import sha512
from functools import wraps
from werkzeug.local import LocalProxy
from werkzeug.security import safe_str_cmp
from werkzeug.urls import url_decode, url_encode

~~from flask import (_request_ctx_stack, current_app, request, session, url_for,
                   has_request_context)

from ._compat import text_type, urlparse, urlunparse
from .config import COOKIE_NAME, EXEMPT_METHODS
from .signals import user_logged_in, user_logged_out, user_login_confirmed


current_user = LocalProxy(lambda: _get_user())


def encode_cookie(payload, key=None):
    return u'{0}|{1}'.format(payload, _cookie_digest(payload, key=key))


def decode_cookie(cookie, key=None):
    try:
        payload, digest = cookie.rsplit(u'|', 1)
        if hasattr(digest, 'decode'):
            digest = digest.decode('ascii')  # pragma: no cover
    except ValueError:
        return

    if safe_str_cmp(_cookie_digest(payload, key=key), digest):
        return payload


## ... source file abbreviated to get to current_app examples ...



def expand_login_view(login_view):
    if login_view.startswith(('https://', 'http://', '/')):
        return login_view
    else:
        if request.view_args is None:
            return url_for(login_view)
        else:
            return url_for(login_view, **request.view_args)


def login_url(login_view, next_url=None, next_field='next'):
    base = expand_login_view(login_view)

    if next_url is None:
        return base

    parsed_result = urlparse(base)
    md = url_decode(parsed_result.query)
    md[next_field] = make_next_param(base, next_url)
~~    netloc = current_app.config.get('FORCE_HOST_FOR_REDIRECTS') or \
        parsed_result.netloc
    parsed_result = parsed_result._replace(netloc=netloc,
                                           query=url_encode(md, sort=True))
    return urlunparse(parsed_result)


def login_fresh():
    return session.get('_fresh', False)


def login_user(user, remember=False, duration=None, force=False, fresh=True):
    if not force and not user.is_active:
        return False

~~    user_id = getattr(user, current_app.login_manager.id_attribute)()
    session['_user_id'] = user_id
    session['_fresh'] = fresh
~~    session['_id'] = current_app.login_manager._session_identifier_generator()

    if remember:
        session['_remember'] = 'set'
        if duration is not None:
            try:
                session['_remember_seconds'] = (duration.microseconds +
                                                (duration.seconds +
                                                 duration.days * 24 * 3600) *
                                                10**6) / 10.0**6
            except AttributeError:
                raise Exception('duration must be a datetime.timedelta, '
                                'instead got: {0}'.format(duration))

~~    current_app.login_manager._update_request_context_with_user(user)
    user_logged_in.send(current_app._get_current_object(), user=_get_user())
    return True


def logout_user():

    user = _get_user()

    if '_user_id' in session:
        session.pop('_user_id')

    if '_fresh' in session:
        session.pop('_fresh')

    if '_id' in session:
        session.pop('_id')

~~    cookie_name = current_app.config.get('REMEMBER_COOKIE_NAME', COOKIE_NAME)
    if cookie_name in request.cookies:
        session['_remember'] = 'clear'
        if '_remember_seconds' in session:
            session.pop('_remember_seconds')

    user_logged_out.send(current_app._get_current_object(), user=user)

~~    current_app.login_manager._update_request_context_with_user()
    return True


def confirm_login():
    session['_fresh'] = True
~~    session['_id'] = current_app.login_manager._session_identifier_generator()
    user_login_confirmed.send(current_app._get_current_object())


def login_required(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.method in EXEMPT_METHODS:
            return func(*args, **kwargs)
~~        elif current_app.config.get('LOGIN_DISABLED'):
            return func(*args, **kwargs)
        elif not current_user.is_authenticated:
~~            return current_app.login_manager.unauthorized()
        return func(*args, **kwargs)
    return decorated_view


def fresh_login_required(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.method in EXEMPT_METHODS:
            return func(*args, **kwargs)
~~        elif current_app.config.get('LOGIN_DISABLED'):
            return func(*args, **kwargs)
        elif not current_user.is_authenticated:
~~            return current_app.login_manager.unauthorized()
        elif not login_fresh():
~~            return current_app.login_manager.needs_refresh()
        return func(*args, **kwargs)
    return decorated_view


def set_login_view(login_view, blueprint=None):

    num_login_views = len(current_app.login_manager.blueprint_login_views)
    if blueprint is not None or num_login_views != 0:

        (current_app.login_manager
            .blueprint_login_views[blueprint.name]) = login_view

        if (current_app.login_manager.login_view is not None and
~~                None not in current_app.login_manager.blueprint_login_views):

            (current_app.login_manager
                .blueprint_login_views[None]) = (current_app.login_manager
                                                 .login_view)

~~        current_app.login_manager.login_view = None
    else:
~~        current_app.login_manager.login_view = login_view


def _get_user():
    if has_request_context() and not hasattr(_request_ctx_stack.top, 'user'):
~~        current_app.login_manager._load_user()

    return getattr(_request_ctx_stack.top, 'user', None)


def _cookie_digest(payload, key=None):
    key = _secret_key(key)

    return hmac.new(key, payload.encode('utf-8'), sha512).hexdigest()


def _get_remote_addr():
    address = request.headers.get('X-Forwarded-For', request.remote_addr)
    if address is not None:
        address = address.encode('utf-8').split(b',')[0].strip()
    return address


def _create_identifier():
    user_agent = request.headers.get('User-Agent')
    if user_agent is not None:
        user_agent = user_agent.encode('utf-8')
    base = '{0}|{1}'.format(_get_remote_addr(), user_agent)
    if str is bytes:
        base = text_type(base, 'utf-8', errors='replace')  # pragma: no cover
    h = sha512()
    h.update(base.encode('utf8'))
    return h.hexdigest()


def _user_context_processor():
    return dict(current_user=_get_user())


def _secret_key(key=None):
    if key is None:
~~        key = current_app.config['SECRET_KEY']

    if isinstance(key, text_type):  # pragma: no cover
        key = key.encode('latin1')  # ensure bytes

    return key



## ... source file continues with no further current_app examples...

```


## Example 7 from flask-restx
[Flask RESTX](https://github.com/python-restx/flask-restx) is an
extension that makes it easier to build
[RESTful APIs](/application-programming-interfaces.html) into
your applications. Flask RESTX aims for minimal configuration to
get basic APIs running for existing applications and it exposes
endpoint documentation using [Swagger](https://swagger.io/).

Flask RESTX is provided as open source under the
[BSD  3-Clause license](https://github.com/python-restx/flask-restx/blob/master/LICENSE).

[**flask-restx / flask_restx / swagger.py**](https://github.com/python-restx/flask-restx/blob/master/flask_restx/./swagger.py)

```python
# swagger.py
from __future__ import unicode_literals, absolute_import

import itertools
import re

from inspect import isclass, getdoc
from collections import OrderedDict

try:
    from collections.abc import Hashable
except ImportError:
    from collections import Hashable
from six import string_types, itervalues, iteritems, iterkeys

~~from flask import current_app
from werkzeug.routing import parse_rule

from . import fields
from .model import Model, ModelBase
from .reqparse import RequestParser
from .utils import merge, not_none, not_none_sorted
from ._http import HTTPStatus

try:
    from urllib.parse import quote
except ImportError:
    from urllib import quote

PATH_TYPES = {
    "int": "integer",
    "float": "number",
    "string": "string",
    "default": "string",
}


PY_TYPES = {
    int: "integer",
    float: "number",


## ... source file abbreviated to get to current_app examples ...


    return {"$ref": "#/definitions/{0}".format(quote(name, safe=""))}


def _v(value):
    return value() if callable(value) else value


def extract_path(path):
    return RE_URL.sub(r"{\1}", path)


def extract_path_params(path):
    params = OrderedDict()
    for converter, arguments, variable in parse_rule(path):
        if not converter:
            continue
        param = {"name": variable, "in": "path", "required": True}

        if converter in PATH_TYPES:
            param["type"] = PATH_TYPES[converter]
~~        elif converter in current_app.url_map.converters:
            param["type"] = "string"
        else:
            raise ValueError("Unsupported type converter: %s" % converter)
        params[variable] = param
    return params


def _param_to_header(param):
    param.pop("in", None)
    param.pop("name", None)
    return _clean_header(param)


def _clean_header(header):
    if isinstance(header, string_types):
        header = {"description": header}
    typedef = header.get("type", "string")
    if isinstance(typedef, Hashable) and typedef in PY_TYPES:
        header["type"] = PY_TYPES[typedef]
    elif (
        isinstance(typedef, (list, tuple))
        and len(typedef) == 1
        and typedef[0] in PY_TYPES
    ):


## ... source file abbreviated to get to current_app examples ...


            }
        if self.api.license:
            infos["license"] = {"name": _v(self.api.license)}
            if self.api.license_url:
                infos["license"]["url"] = _v(self.api.license_url)

        paths = {}
        tags = self.extract_tags(self.api)

        responses = self.register_errors()

        for ns in self.api.namespaces:
            for resource, urls, route_doc, kwargs in ns.resources:
                for url in self.api.ns_urls(ns, urls):
                    path = extract_path(url)
                    serialized = self.serialize_resource(
                        ns, resource, url, route_doc=route_doc, **kwargs
                    )
                    paths[path] = serialized

~~        if current_app.config["RESTX_INCLUDE_ALL_MODELS"]:
            for m in self.api.models:
                self.register_model(m)

        for ns in self.api.namespaces:
            if ns.authorizations:
                if self.api.authorizations is None:
                    self.api.authorizations = {}
                self.api.authorizations = merge(
                    self.api.authorizations, ns.authorizations
                )

        specs = {
            "swagger": "2.0",
            "basePath": basepath,
            "paths": not_none_sorted(paths),
            "info": infos,
            "produces": list(iterkeys(self.api.representations)),
            "consumes": ["application/json"],
            "securityDefinitions": self.api.authorizations or None,
            "security": self.security_requirements(self.api.security) or None,
            "tags": tags,
            "definitions": self.serialize_definitions() or None,
            "responses": responses or None,
            "host": self.get_host(),
        }
        return not_none(specs)

    def get_host(self):
~~        hostname = current_app.config.get("SERVER_NAME", None) or None
        if hostname and self.api.blueprint and self.api.blueprint.subdomain:
            hostname = ".".join((self.api.blueprint.subdomain, hostname))
        return hostname

    def extract_tags(self, api):
        tags = []
        by_name = {}
        for tag in api.tags:
            if isinstance(tag, string_types):
                tag = {"name": tag}
            elif isinstance(tag, (list, tuple)):
                tag = {"name": tag[0], "description": tag[1]}
            elif isinstance(tag, dict) and "name" in tag:
                pass
            else:
                raise ValueError("Unsupported tag format for {0}".format(tag))
            tags.append(tag)
            by_name[tag["name"]] = tag
        for ns in api.namespaces:
            if not ns.resources:
                continue
            if all(is_hidden(r.resource, route_doc=r.route_doc) for r in ns.resources):
                continue
            if ns.name not in by_name:


## ... source file abbreviated to get to current_app examples ...


        for name, param in iteritems(doc["params"]):
            param["name"] = name
            if "type" not in param and "schema" not in param:
                param["type"] = "string"
            if "in" not in param:
                param["in"] = "query"

            if "type" in param and "schema" not in param:
                ptype = param.get("type", None)
                if isinstance(ptype, (list, tuple)):
                    typ = ptype[0]
                    param["type"] = "array"
                    param["items"] = {"type": PY_TYPES.get(typ, typ)}

                elif isinstance(ptype, (type, type(None))) and ptype in PY_TYPES:
                    param["type"] = PY_TYPES[ptype]

            params.append(param)

        mask = doc.get("__mask__")
~~        if mask and current_app.config["RESTX_MASK_SWAGGER"]:
            param = {
~~                "name": current_app.config["RESTX_MASK_HEADER"],
                "in": "header",
                "type": "string",
                "format": "mask",
                "description": "An optional fields mask",
            }
            if isinstance(mask, string_types):
                param["default"] = mask
            params.append(param)

        return params

    def responses_for(self, doc, method):
        responses = {}

        for d in doc, doc[method]:
            if "responses" in d:
                for code, response in iteritems(d["responses"]):
                    code = str(code)
                    if isinstance(response, string_types):
                        description = response
                        model = None
                        kwargs = {}
                    elif len(response) == 3:
                        description, model, kwargs = response


## ... source file continues with no further current_app examples...

```


## Example 8 from flask-sqlalchemy
[flask-sqlalchemy](https://github.com/pallets/flask-sqlalchemy)
([project documentation](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)
and
[PyPI information](https://pypi.org/project/Flask-SQLAlchemy/)) is a
[Flask](/flask.html) extension that makes it easier to use
[SQLAlchemy](/sqlalchemy.html) when building Flask apps. flask-sqlalchemy
provides helper functions that reduce the amount of common boilerplate
code that you have to frequently write yourself if you did not use this
library when combining Flask with SQLAlchemy.

flask-sqlalchemy is provided as open source under the
[BSD 3-Clause "New" or "Revised" License](https://github.com/pallets/flask-sqlalchemy/blob/master/LICENSE.rst).

[**flask-sqlalchemy / src/flask_sqlalchemy / __init__.py**](https://github.com/pallets/flask-sqlalchemy/blob/master/src/flask_sqlalchemy/./__init__.py)

```python
# __init__.py
import functools
import os
import sys
import warnings
from math import ceil
from operator import itemgetter
from threading import Lock
from time import perf_counter

import sqlalchemy
from flask import _app_ctx_stack
from flask import abort
~~from flask import current_app
from flask import request
from flask.signals import Namespace
from sqlalchemy import event
from sqlalchemy import inspect
from sqlalchemy import orm
from sqlalchemy.engine.url import make_url
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.ext.declarative import DeclarativeMeta
from sqlalchemy.orm.exc import UnmappedClassError
from sqlalchemy.orm.session import Session as SessionBase

from .model import DefaultMeta
from .model import Model

__version__ = "3.0.0.dev"

_signals = Namespace()
models_committed = _signals.signal("models-committed")
before_models_committed = _signals.signal("before-models-committed")


def _make_table(db):
    def _make_table(*args, **kwargs):
        if len(args) > 1 and isinstance(args[1], db.Column):


## ... source file abbreviated to get to current_app examples ...


        state = get_state(app)

        with self._engine_lock:
            connector = state.connectors.get(bind)

            if connector is None:
                connector = self.make_connector(app, bind)
                state.connectors[bind] = connector

            return connector.get_engine()

    def create_engine(self, sa_url, engine_opts):
        return sqlalchemy.create_engine(sa_url, **engine_opts)

    def get_app(self, reference_app=None):

        if reference_app is not None:
            return reference_app

        if current_app:
~~            return current_app._get_current_object()

        if self.app is not None:
            return self.app

        raise RuntimeError(
            "No application found. Either work inside a view function or push"
            " an application context. See"
            " http://flask-sqlalchemy.pocoo.org/contexts/."
        )

    def get_tables_for_bind(self, bind=None):
        result = []
        for table in self.Model.metadata.tables.values():
            if table.info.get("bind_key") == bind:
                result.append(table)
        return result

    def get_binds(self, app=None):
        app = self.get_app(app)
        binds = [None] + list(app.config.get("SQLALCHEMY_BINDS") or ())
        retval = {}
        for bind in binds:
            engine = self.get_engine(app, bind)
            tables = self.get_tables_for_bind(bind)


## ... source file continues with no further current_app examples...

```


## Example 9 from Flask-WTF
[Flask-WTF](https://github.com/lepture/flask-wtf)
([project documentation](https://flask-wtf.readthedocs.io/en/stable/)
and
[PyPI page](https://pypi.org/project/Flask-WTF/))
provides a bridge between [Flask](/flask.html) and the the
[WTForms](https://wtforms.readthedocs.io/en/2.3.x/) form-handling library.
It makes it easier to use WTForms by reducing boilerplate code and
shorter examples for common form operations as well as common security
practices such as [CSRF](/cross-site-request-forgery-csrf.html).

[**Flask-WTF / flask_wtf / csrf.py**](https://github.com/lepture/flask-wtf/blob/master/flask_wtf/./csrf.py)

```python
# csrf.py
import hashlib
import logging
import os
import warnings
from functools import wraps

~~from flask import Blueprint, current_app, g, request, session
from itsdangerous import BadData, SignatureExpired, URLSafeTimedSerializer
from werkzeug.exceptions import BadRequest
from werkzeug.security import safe_str_cmp
from wtforms import ValidationError
from wtforms.csrf.core import CSRF

from ._compat import FlaskWTFDeprecationWarning, string_types, urlparse

__all__ = ('generate_csrf', 'validate_csrf', 'CSRFProtect')
logger = logging.getLogger(__name__)


def generate_csrf(secret_key=None, token_key=None):

    secret_key = _get_config(
~~        secret_key, 'WTF_CSRF_SECRET_KEY', current_app.secret_key,
        message='A secret key is required to use CSRF.'
    )
    field_name = _get_config(
        token_key, 'WTF_CSRF_FIELD_NAME', 'csrf_token',
        message='A field name is required to use CSRF.'
    )

    if field_name not in g:
        s = URLSafeTimedSerializer(secret_key, salt='wtf-csrf-token')

        if field_name not in session:
            session[field_name] = hashlib.sha1(os.urandom(64)).hexdigest()

        try:
            token = s.dumps(session[field_name])
        except TypeError:
            session[field_name] = hashlib.sha1(os.urandom(64)).hexdigest()
            token = s.dumps(session[field_name])

        setattr(g, field_name, token)

    return g.get(field_name)


def validate_csrf(data, secret_key=None, time_limit=None, token_key=None):

    secret_key = _get_config(
~~        secret_key, 'WTF_CSRF_SECRET_KEY', current_app.secret_key,
        message='A secret key is required to use CSRF.'
    )
    field_name = _get_config(
        token_key, 'WTF_CSRF_FIELD_NAME', 'csrf_token',
        message='A field name is required to use CSRF.'
    )
    time_limit = _get_config(
        time_limit, 'WTF_CSRF_TIME_LIMIT', 3600, required=False
    )

    if not data:
        raise ValidationError('The CSRF token is missing.')

    if field_name not in session:
        raise ValidationError('The CSRF session token is missing.')

    s = URLSafeTimedSerializer(secret_key, salt='wtf-csrf-token')

    try:
        token = s.loads(data, max_age=time_limit)
    except SignatureExpired:
        raise ValidationError('The CSRF token has expired.')
    except BadData:
        raise ValidationError('The CSRF token is invalid.')

    if not safe_str_cmp(session[field_name], token):
        raise ValidationError('The CSRF tokens do not match.')


def _get_config(
    value, config_name, default=None,
    required=True, message='CSRF is not configured.'
):

    if value is None:
~~        value = current_app.config.get(config_name, default)

    if required and value is None:
        raise RuntimeError(message)

    return value


class _FlaskFormCSRF(CSRF):
    def setup_form(self, form):
        self.meta = form.meta
        return super(_FlaskFormCSRF, self).setup_form(form)

    def generate_csrf_token(self, csrf_token_field):
        return generate_csrf(
            secret_key=self.meta.csrf_secret,
            token_key=self.meta.csrf_field_name
        )

    def validate_csrf_token(self, form, field):
        if g.get('csrf_valid', False):
            return

        try:
            validate_csrf(


## ... source file abbreviated to get to current_app examples ...


                return

            if request.method not in app.config['WTF_CSRF_METHODS']:
                return

            if not request.endpoint:
                return

            if request.blueprint in self._exempt_blueprints:
                return

            view = app.view_functions.get(request.endpoint)
            dest = '{0}.{1}'.format(view.__module__, view.__name__)

            if dest in self._exempt_views:
                return

            self.protect()

    def _get_csrf_token(self):
~~        field_name = current_app.config['WTF_CSRF_FIELD_NAME']
        base_token = request.form.get(field_name)

        if base_token:
            return base_token

        for key in request.form:
            if key.endswith(field_name):
                csrf_token = request.form[key]

                if csrf_token:
                    return csrf_token

~~        for header_name in current_app.config['WTF_CSRF_HEADERS']:
            csrf_token = request.headers.get(header_name)

            if csrf_token:
                return csrf_token

        return None

    def protect(self):
~~        if request.method not in current_app.config['WTF_CSRF_METHODS']:
            return

        try:
            validate_csrf(self._get_csrf_token())
        except ValidationError as e:
            logger.info(e.args[0])
            self._error_response(e.args[0])

~~        if request.is_secure and current_app.config['WTF_CSRF_SSL_STRICT']:
            if not request.referrer:
                self._error_response('The referrer header is missing.')

            good_referrer = 'https://{0}/'.format(request.host)

            if not same_origin(request.referrer, good_referrer):
                self._error_response('The referrer does not match the host.')

        g.csrf_valid = True  # mark this request as CSRF valid

    def exempt(self, view):

        if isinstance(view, Blueprint):
            self._exempt_blueprints.add(view.name)
            return view

        if isinstance(view, string_types):
            view_location = view
        else:
            view_location = '.'.join((view.__module__, view.__name__))

        self._exempt_views.add(view_location)
        return view

    def _error_response(self, reason):
        raise CSRFError(reason)

    def error_handler(self, view):

        warnings.warn(FlaskWTFDeprecationWarning(
            '"@csrf.error_handler" is deprecated. Use the standard Flask '
            'error system with "@app.errorhandler(CSRFError)" instead. This '
            'will be removed in 1.0.'
        ), stacklevel=2)

        @wraps(view)
        def handler(reason):
~~            response = current_app.make_response(view(reason))
            raise CSRFError(response=response)

        self._error_response = handler
        return view


class CsrfProtect(CSRFProtect):

    def __init__(self, app=None):
        warnings.warn(FlaskWTFDeprecationWarning(
            '"flask_wtf.CsrfProtect" has been renamed to "CSRFProtect" '
            'and will be removed in 1.0.'
        ), stacklevel=2)
        super(CsrfProtect, self).__init__(app=app)


class CSRFError(BadRequest):

    description = 'CSRF validation failed.'


def same_origin(current_uri, compare_uri):
    current = urlparse(current_uri)
    compare = urlparse(compare_uri)


## ... source file continues with no further current_app examples...

```


## Example 10 from Flasky
[Flasky](https://github.com/miguelgrinberg/flasky) is a wonderful
example application by
[Miguel Grinberg](https://github.com/miguelgrinberg) that he builds
while teaching developers how to use [Flask](/flask.html) in
[his books and videos](https://courses.miguelgrinberg.com/). Flasky
is [open sourced under the MIT license](https://github.com/miguelgrinberg/flasky/blob/master/LICENSE).

[**Flasky / migrations / env.py**](https://github.com/miguelgrinberg/flasky/blob/master/./migrations/env.py)

```python
# env.py
from __future__ import with_statement
from alembic import context
from sqlalchemy import engine_from_config, pool
from logging.config import fileConfig

config = context.config

fileConfig(config.config_file_name)

~~from flask import current_app
~~config.set_main_option('sqlalchemy.url', current_app.config.get('SQLALCHEMY_DATABASE_URI'))
~~target_metadata = current_app.extensions['migrate'].db.metadata


def run_migrations_offline():
    url = config.get_main_option("sqlalchemy.url")
    context.configure(url=url)

    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    engine = engine_from_config(
                config.get_section(config.config_ini_section),
                prefix='sqlalchemy.',
                poolclass=pool.NullPool)

    connection = engine.connect()
    context.configure(
                connection=connection,
                target_metadata=target_metadata
                )

    try:
        with context.begin_transaction():
            context.run_migrations()


## ... source file continues with no further current_app examples...

```


## Example 11 from sandman2
[sandman2](https://github.com/jeffknupp/sandman2)
([project documentation](https://sandman2.readthedocs.io/en/latest/)
and
[PyPI package information](https://pypi.org/project/sandman2/))
is a code library for automatically generating
[RESTful APIs](/application-programming-interfaces.html) from
existing database schemas. This approach is handy for solving
straightforward situations where you want to put an abstraction
layer between one or more applications and your
[relational database](/databases.html) to prevent or reduce
direct database access.

The sandman2 project is provided under the
[Apache License 2.0](https://github.com/jeffknupp/sandman2/blob/master/LICENSE).

[**sandman2 / sandman2 / app.py**](https://github.com/jeffknupp/sandman2/blob/master/sandman2/./app.py)

```python
# app.py

~~from flask import Flask, current_app, jsonify
from sqlalchemy.sql import sqltypes

from sandman2.exception import (
    BadRequestException,
    ForbiddenException,
    NotFoundException,
    NotAcceptableException,
    NotImplementedException,
    ConflictException,
    ServerErrorException,
    ServiceUnavailableException,
    )
from sandman2.service import Service
from sandman2.model import db, Model, AutomapModel
from sandman2.admin import CustomAdminView
from flask_admin import Admin
from flask_httpauth import HTTPBasicAuth

auth = HTTPBasicAuth()

def get_app(
        database_uri,
        exclude_tables=None,
        user_models=None,


## ... source file abbreviated to get to current_app examples ...


def _register_error_handlers(app):
    @app.errorhandler(BadRequestException)
    @app.errorhandler(ForbiddenException)
    @app.errorhandler(NotAcceptableException)
    @app.errorhandler(NotFoundException)
    @app.errorhandler(ConflictException)
    @app.errorhandler(ServerErrorException)
    @app.errorhandler(NotImplementedException)
    @app.errorhandler(ServiceUnavailableException)
    def handle_application_error(error):  # pylint:disable=unused-variable
        response = jsonify(error.to_dict())
        response.status_code = error.code
        return response


def register_service(cls, primary_key_type):
    view_func = cls.as_view(cls.__name__.lower())  # pylint: disable=no-member
    methods = set(cls.__model__.__methods__)  # pylint: disable=no-member

    if 'GET' in methods:  # pylint: disable=no-member
~~        current_app.add_url_rule(
            cls.__model__.__url__ + '/', defaults={'resource_id': None},
            view_func=view_func,
            methods=['GET'])
~~        current_app.add_url_rule(
            '{resource}/meta'.format(resource=cls.__model__.__url__),
            view_func=view_func,
            methods=['GET'])
    if 'POST' in methods:  # pylint: disable=no-member
~~        current_app.add_url_rule(
            cls.__model__.__url__ + '/', view_func=view_func, methods=['POST', ])
~~    current_app.add_url_rule(
        '{resource}/<{pk_type}:{pk}>'.format(
            resource=cls.__model__.__url__,
            pk='resource_id', pk_type=primary_key_type),
        view_func=view_func,
        methods=methods - {'POST'})
~~    current_app.classes.append(cls)


def _reflect_all(exclude_tables=None, admin=None, read_only=False, schema=None):
    AutomapModel.prepare(  # pylint:disable=maybe-no-member
        db.engine, reflect=True, schema=schema)
    for cls in AutomapModel.classes:
        if exclude_tables and cls.__table__.name in exclude_tables:
            continue
        if read_only:
            cls.__methods__ = {'GET'}
        register_model(cls, admin)


def register_model(cls, admin=None):
    cls.__url__ = '/{}'.format(cls.__name__.lower())
    service_class = type(
        cls.__name__ + 'Service',
        (Service,),
        {
            '__model__': cls,
        })

    cols = list(cls().__table__.primary_key.columns)



## ... source file continues with no further current_app examples...

```


## Example 12 from tedivms-flask
[tedivm's flask starter app](https://github.com/tedivm/tedivms-flask) is a
base of [Flask](/flask.html) code and related projects such as
[Celery](/celery.html) which provides a template to start your own
Flask web app. The project comes baked with an admin panel,
[API authentication and authorization](/application-programming-interfaces.html),
[SQLAlchemy](/sqlalchemy.html) and many other common libraries that are
often used with Flask.

The project's code is provided as open source under the
[BSD 2-Clause "Simplified" license](https://github.com/tedivm/tedivms-flask/blob/master/LICENSE.txt).

[**tedivms-flask / app / utils / api.py**](https://github.com/tedivm/tedivms-flask/blob/master/app/utils/api.py)

```python
# api.py
from app.models import user_models as users
from functools import wraps
~~from flask import request, abort, current_app


def is_authorized_api_user(roles=False):
    if 'API_ID' not in request.headers:
        return False
    if 'API_KEY' not in request.headers:
        return False
    api_key = users.ApiKey.query.filter(users.ApiKey.id==request.headers['API_ID']).first()
    if not api_key:
        return False
~~    if not current_app.user_manager.verify_password(request.headers['API_KEY'], api_key.hash):
        return False
    if not roles:
        return True
    if api_key.user.has_role('admin'):
        return True
    for role in roles:
        if api_key.user.has_role(role):
            return True
    return False


def roles_accepted_api(*role_names):
    def wrapper(view_function):
        @wraps(view_function)
        def decorated_view_function(*args, **kwargs):
            if not is_authorized_api_user(role_names):
                return abort(403)
            return view_function(*args, **kwargs)
        return decorated_view_function
    return wrapper


def api_credentials_required():
    def wrapper(view_function):


## ... source file continues with no further current_app examples...

```

