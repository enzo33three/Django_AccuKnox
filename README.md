# Django_AccuKnox


**Assignment Answers: Accuknox Django Trainee**

---

**Topic: Django Signals**

**Question 1: Are Django signals executed synchronously or asynchronously by default?**

**Answer:**
By default, Django signals are executed **synchronously**. This means that the signal handler runs in the same process and blocks the execution of the next line until it finishes.

**Proof (Code Snippet):**
```python
# signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
import time
import logging

logger = logging.getLogger(__name__)

@receiver(post_save, sender=User)
def user_saved_handler(sender, instance, **kwargs):
    logger.info("Signal handler started")
    time.sleep(5)
    logger.info("Signal handler finished")
```

```python
# views.py
from django.contrib.auth.models import User
from django.http import JsonResponse
import logging

logger = logging.getLogger(__name__)

def create_user_view(request):
    logger.info("View start")
    User.objects.create(username='sync_test')
    logger.info("View end")
    return JsonResponse({"status": "done"})
```

**Observation:** You will see in logs:
```
View start
Signal handler started
Signal handler finished
View end
```
This proves the signal is blocking and synchronous.

---

**Question 2: Do Django signals run in the same thread as the caller?**

**Answer:**
Yes, by default Django signals run in the **same thread** as the caller.

**Proof (Code Snippet):**
```python
# signals.py
import threading
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def user_saved_handler(sender, instance, **kwargs):
    print("Signal Thread:", threading.get_ident())
```

```python
# views.py
from django.contrib.auth.models import User
from django.http import JsonResponse
import threading

def create_user_view(request):
    print("View Thread:", threading.get_ident())
    User.objects.create(username='thread_test')
    return JsonResponse({"status": "done"})
```

**Observation:**
The output of both print statements will show the same thread ID, proving they are in the same thread.

---

**Question 3: Do Django signals run in the same database transaction as the caller?**

**Answer:**
Yes, Django signals (like `post_save`) run in the **same transaction** as the caller unless explicitly handled otherwise.

**Proof (Code Snippet):**
```python
# signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.db import connection
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def user_saved_handler(sender, instance, **kwargs):
    print("Signal DB Connection In Transaction:", connection.in_atomic_block)
```

```python
# views.py
from django.contrib.auth.models import User
from django.http import JsonResponse
from django.db import transaction

def create_user_view(request):
    with transaction.atomic():
        print("View In Transaction:", transaction.get_connection().in_atomic_block)
        User.objects.create(username='db_test')
    return JsonResponse({"status": "done"})
```

**Observation:**
Both the view and the signal print `True`, confirming they are within the same transaction.

---

**Topic: Custom Classes in Python**

**Rectangle Class Implementation:**
```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {"length": self.length}
        yield {"width": self.width}

# Example usage:
rect = Rectangle(10, 5)
for item in rect:
    print(item)
```

**Output:**
```
{'length': 10}
{'width': 5}
```

---

**End of Assignment**
