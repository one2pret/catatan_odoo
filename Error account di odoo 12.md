## Pada account invoice odoo 12 mengalami error seperti dibawah ini :
Error:
Odoo Server Error

Traceback (most recent call last):
File "E:\project\odoo2024\odoo12\odoo\http.py", line 663, in _handle_exception
return super(JsonRequest, self)._handle_exception(exception)
File "E:\project\odoo2024\odoo12\odoo\http.py", line 321, in _handle_exception
raise pycompat.reraise(type(exception), exception, sys.exc_info()[2])
File "E:\project\odoo2024\odoo12\odoo\tools\pycompat.py", line 87, in reraise
raise value
File "E:\project\odoo2024\odoo12\odoo\http.py", line 705, in dispatch
result = self._call_function(**self.params)
File "E:\project\odoo2024\odoo12\odoo\http.py", line 353, in _call_function
return checked_call(self.db, *args, **kwargs)
File "E:\project\odoo2024\odoo12\odoo\service\model.py", line 98, in wrapper
return f(dbname, *args, **kwargs)
File "E:\project\odoo2024\odoo12\odoo\http.py", line 346, in checked_call
result = self.endpoint(*a, **kw)
File "E:\project\odoo2024\odoo12\odoo\http.py", line 948, in __call__
return self.method(*args, **kw)
File "E:\project\odoo2024\odoo12\odoo\http.py", line 526, in response_wrap
response = f(*args, **kw)
File "e:\project\odoo2024\odoo12\addons\web\controllers\main.py", line 959, in call_kw
return self._call_kw(model, method, args, kwargs)
File "e:\project\odoo2024\odoo12\addons\web\controllers\main.py", line 951, in _call_kw
return call_kw(request.env[model], method, args, kwargs)
File "E:\project\odoo2024\odoo12\odoo\api.py", line 760, in call_kw
return _call_kw_multi(method, model, args, kwargs)
File "E:\project\odoo2024\odoo12\odoo\api.py", line 747, in _call_kw_multi
result = method(recs, *args, **kwargs)
File "E:\project\odoo2024\odoo12\odoo\models.py", line 2821, in read
self._read_from_database(stored, inherited)
File "E:\project\odoo2024\odoo12\odoo\models.py", line 2948, in _read_from_database
cr.execute(query_str, params)
File "E:\project\odoo2024\odoo12\odoo\sql_db.py", line 148, in wrapper
return f(self, *args, **kwargs)
File "E:\project\odoo2024\odoo12\odoo\sql_db.py", line 225, in execute
res = self._obj.execute(query, params)
psycopg2.errors.UndefinedColumn: column account_invoice_tax.amount_total does not exist
LINE 1: ...voice_tax"."amount_rounding" as "amount_rounding","account_i...
^


### Dapat di atasi dengan mengupdate dulu modul seperti command di bawah ini 
* python3 odoo-bin -d your_database_name -u account --stop-after-init
* E:\project\odoo2024\Master\python36\python.exe  odoo-bin -c .\odoo_config.conf -d klj_16_02_25 -u account

