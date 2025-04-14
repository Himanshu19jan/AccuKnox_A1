# AccuKnox_Django_Assignment
______________________________________________________________________________________________________________________________________
**Q1.** Are Django Signals Executed Synchronously or Asynchronously by Default?
_______________________________________________________________________________________________________________________________________
By default, Django signals are executed synchronously. This means when a signal is triggered (e.g., post_save, pre_save), all connected receiver functions are called immediately in the same thread as the operation that emitted the signal. The main execution flow waits for the signal handlers to complete before continuing.

This behavior is crucial to understand because if a signal handler performs a long-running or blocking operation (e.g., sending emails, logging, calling APIs), it can negatively impact performance and responsiveness of the application — especially in views or transactional logic.

Why is it synchronous?
Django’s signal system is built using the blinker or dispatch system which follows the observer pattern. When a signal is emitted, it loops through all connected receivers and executes them one by one. This is done in the same thread without spawning a new process or background thread unless explicitly coded.

----------------------------------------------------------------------
**File - models.py**

from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver
import time

class Book(models.Model):
    title = models.CharField(max_length=100)

@receiver(post_save, sender=Book)
def on_book_saved(sender, instance, created, **kwargs):
    print("Signal started")
    time.sleep(3)  # Simulate long task
    print("Signal finished")

-----------------------------------------------------------------------

**view.py**

from django.http import HttpResponse
from .models import Book
import time

def create_book_view(request):
    start = time.time()
    Book.objects.create(title="Signal Test")
    end = time.time()
    return HttpResponse(f"Book created in {end - start:.2f} seconds")
__________________________________________________________________________________________________________________________________________
**Q2**. Do django signals run in the same thread as the caller?
__________________________________________________________________________________________________________________________________________


Yes, Django signals run in the same thread as the caller by default. There is no thread-switching unless you explicitly introduce threading or asynchronous behavior yourself.
Let’s use threading.get_ident() to log the thread IDs during the signal and the view execution.

---------------------------------------------------------------------------------
**models.py**
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver
import threading

class Book(models.Model):
    title = models.CharField(max_length=100)

@receiver(post_save, sender=Book)
def book_saved_signal(sender, instance, **kwargs):
    print(f"[Signal] Running in thread: {threading.get_ident()}")

---------------------------------------------------------------------------------
**views.py**

from django.http import HttpResponse
from .models import Book
import threading

def create_book_view(request):
    print(f"[View] Running in thread: {threading.get_ident()}")
    Book.objects.create(title="Thread Test")
    return HttpResponse("Book created")

--------------------------------------------------------------------------------
Expected Output: 
[View] Running in thread: 140735224853376
[Signal] Running in thread: 140735224853376

__________________________________________________________________________________________________________________________________________
**Q3.** By default do django signals run in the same database transaction as the caller?
___________________________________________________________________________________________________________________________________________


Yes, by default, Django signals run in the same database transaction as the caller.

This means:
If a post_save signal is triggered during a .save() or .create() call inside a transaction,
And if that outer transaction is rolled back,
Any changes made inside the signal handler will also be rolled back.

Code Snippet to Prove It
We will:
Use transaction.atomic() to start a transaction.
Create a model object to trigger the signal.
Intentionally raise an exception after the save.
Observe that both the original save and any DB writes inside the signal are rolled back.

---------------------------------------------------------------------------------
**models.py**

from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver

class Book(models.Model):
    title = models.CharField(max_length=100)

class Log(models.Model):
    message = models.CharField(max_length=100)

@receiver(post_save, sender=Book)
def book_post_save(sender, instance, **kwargs):
    Log.objects.create(message=f"Book created with ID {instance.id}")

------------------------------------------------------------------------------------
**view.py**

from django.http import HttpResponse
from django.db import transaction
from .models import Book, Log

def create_book_view(request):
    try:
        with transaction.atomic():
            Book.objects.create(title="Transactional Test")
            raise Exception("Force rollback!")
    except:
        pass

    # Check if any logs were created
    logs = Log.objects.all()
    return HttpResponse(f"Log count: {logs.count()}")

_________________________________________________________________________________________________________________________________________
Topic: Custom Classes in Python - creating a Rectangle class
_________________________________________________________________________________________________________________________________________
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {'length': self.length}
        yield {'width': self.width}
------------------------------------------------------------------------------
Example - 
rect = Rectangle(10, 5)
for item in rect:
    print(item)

Output: 
{'length': 10}
{'width': 5}

