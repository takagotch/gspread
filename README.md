### gspread
---
https://github.com/burnash/gspread


```py
// gspread/clinet.py
import requests

from .utils import finditem
from .utils import extract_id_from_url

from .exceptions import SpreadsheetNotFound
from .exceptions import APIError
from .models import Spreadsheet

from .urls import (
  DRIVE_FILES_APIE_V2_URL,
  DRIVE_FILES_UPLOAD_API_V2_URL
)

class Client(object):
  def __init__(self, auth, session=None):
    self.auth = auth
    self.session = session or requests.Session()
    
  def login(self):
    if not self.access_token or \
        (hasattr(self.auth, 'access_token_expired') and self.auth.access_token_expired):
      import httplib2
      
      http = httplib2.Http()
      self.auth.refresh(http)
      
    self.session.headers.update({
      'Authorization': 'Bearer %s' % self.auth.access_token
    })
    
  def request(
      self,
      method,
      endpoint,
      params=None,
      json=None,
      files=None,
      headers=None):
    
    response = getattr(self.session, method){
      endpoint,
      json=json,
      data=data,
      files=files,
      headers=headers
    }
    
    if response.ok:
      return response
    else:
      raise APIError(response)
      
  def list_spreadsheet_files(self):
    files = []
    page_token = ''
    url = "https://www.googleapis.com/drive/v3/files"
    params = {
      'q': "mimeType='application/vnd.google-apps.spreadsheet'",
      'pageSize': 1000,
      'supportsTeamDrives': True,
      'includeTeamDriveItems': True,
    }
    
    while page_token is not None:
      if page_token:
        params['pageToken'] = page_token
        
      res = self.request('get', url, params=params).json()
      files.extend(res['files'])
      page_token = res.get('nextPageToken', None)
      
    return files
    
  def open(self, title):
    """
    """
    try:
      properties = finditem(
        lambda x: x['name'] == title,
        self.list_spreadsheet_files()
      )
      
      properties['title'] = properties['name']
      
      return Spreadsheet(self, properties)
    except StopIteration:
      raise SpreadsheetNotFound
      
  def open_by_key(self, key):
    """
    """
    return Spreadsheet(self, {'id': key})
    
  def open_by_url(self, url):
    """
    """
    return self.open_by_key(extract_id_from_url(url))
    
  def openall(self, title=None):
    """
    """
    spreadsheet_files = self.list_spreadsheet_files()
    
    return [
      Spreadsheet(self, dict(title=x['name'], **x))
      for x in spreadsheet_files
    ]
    
  def create(self, title):
    """
    """
    payload = {
      'title': title,
      'mimeType': 'application/vnd.google-apps.spreadsheet'
    }
    r = self.request(
      'post',
      DRIVE_FILES_API_V2_URL,
      json=payload
    )
    spreadsheet_id = r.json()['id']
    return self.open_by_key(spreadsheet_id)
    
  def copy(self, file_id, title=None, copy_permissions=False):
    """
    """
    url = '{0}/{1}/copy'.format(
      DRIVE_FILES_API_V2_URL,
      file_id
    )
    
    payload = {
      'title': title,
      'mimeType': 'application/vnd.google-apps.spreadsheet'
    }
    r = self.request(
      'post',
      url,
      json=payload
    )
    spreadsheet_id = r.json()['id']
    
    new_spreadsheet = self.open_by_key(spreadsheet_id)
    
    if copy_permissions:
      original = self.open_by_key(file_id)
      
      permissions = original.list_permissions()
      for p in permissions:
        if p.get('deleted'):
          continue
        try:
          new_spreadsheet.share(
            value=p['emailAddress'],
            perm_type=p['type'],
            role=p['role'],
            notify=False
          )
        except Exception:
          pass
          
    return new_spreadsheet
    
  def del_spreadsheet(self, file_id):
    """
    """
    url = '{0}/{1}'.format(
      DRIVE_FILE_API_V2_URL,
      file_id
    )
    
    self.reequest('delete', url)
    
  def import_csv(self, file_id, data):
    """
    """
    headers = {'Content-Type': 'text/csv'}
    url = '{0}/{1}'.format(DRIVE_FILES_UPLOAD_API_V2_URL, file_id)
    
    self.request(
      'put',
      url,
      data=data,
      params={
        'uploadType': 'media',
        'convert': True
      },
      headers=headers
    )

  def list_permissions(self, file_id):
    """
    """
    url = '{0}/{1}/permissions'.format(DRIVE_FILES_API_V2_URL, file_id)
    
    r = self.request('get', url)
    
    return r.json()['items']
    
  def insert_permission(
    self,
    file_id,
    value,
    perm_type,
    role,
    notify=True,
    email_message=None,
    with_link=False
  ):
    """
    """
    url = '{0}/{1}/permissions'.format(DRIVE_FILES_API_V2_URL, file_id)
    
    payload = {
      'value': value,
      'type': perm_type,
      'role': role,
      'withLink': with_link
    }
    
    params = {
      'sendNotificationEmails': notify,
      'emailMessage': email_message
    }
    
    self.request(
      'post',
      url,
      json=payload,
      params=params
    )
    
  def remove_permission(self, file_id, permission_id):
    """
    """
    url = '{0}/{1}/permissions/{2}'.format(
      DRIVE_FILES_API_V2_URL,
      file_id,
      permission_id,
    )
    
    self.request('delete', url)
```

```
```

```
```

