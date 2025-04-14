# AccuKnox_A1

**Q1.** Are Django Signals Executed Synchronously or Asynchronously by Default?
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

------------------------------------------------------------------------
