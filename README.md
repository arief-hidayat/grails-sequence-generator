# Grails Sequence Generator Plugin

The sequence generator plugin provides a simple way to add sequence counters
to Grails applications. You can control the starting number, the format and
you can have different sequence counters based on application logic.

**Example**

    sequenceService.initSequence('WebOrder', null, null, 100, 'WEB-%04d')

    assert sequenceGeneratorService.nextNumber('WebOrder') == 'WEB-0100'
    assert sequenceGeneratorService.nextNumber('WebOrder') == 'WEB-0101'
    assert sequenceGeneratorService.nextNumber('WebOrder') == 'WEB-0102'

    assert sequenceGeneratorService.nextNumberLong('WebOrder') == 103
    assert sequenceGeneratorService.nextNumberLong('WebOrder') == 104

The SequenceGeneratorService implementation is very efficient and can provide
sequential numbers to concurrent threads without problems.

Sequences are persisted to database to survive server restarts.

Domain classes annotated with grails.plugins.sequence.SequenceEntity
will get a 'number' property added at compile time and a getNextSequenceNumber() method at runtime.

## Configuration

**sequence.flushInterval** (default 60)

Number of seconds to wait before flushing in-memory sequence counters to disk.

    sequence.flushInterval = 300
    
**sequence.<name>.format** (default %d)

Format to use for sequence numbers for a domain class. The name is the simple name of the domain class, without package.
The number is formatted with *String#format(String, Object...)*.

    sequence.Customer.format = "%05d"

**sequence.<name>.start** (default 1)

The starting sequence for a domain class. The name is the simple name of the domain class, without package.

    sequence.Customer.start = 1001


## SequenceGeneratorService

**def initSequence(Class clazz, String group = null, Long tenant = null, Long start = null, String format = null)**

Initialize/create a new sequence counter for a domain class.

Parameter   | Description
----------- | ---------------------
clazz       | Domain class to generate sequences for
group       | If you need multiple sequences for the same domain class based on some application logic you can use groups to create sub-sequences
tenant      | Tenant ID in a multi-tenant environment
start       | The sequence will start at this number
format      | The number format returned by *nextNumber()* uses String#format(String, Object...)

**String nextNumber(Class clazz, String group = null, Long tenant = null)**

Returns the next number in the specified sequence.

Parameter   | Description
----------- | ---------------------
clazz       | Domain class to get sequence for
group       | Optional sub-sequence if multiple sequence counters exists for the domain class
tenant      | Tenant ID in a multi-tenant environment

**boolean setNextNumber(Long currentNumber, Long newNumber, String name, String group = null, Long tenant = null)**

Sets the next number for a sequence counter.
To avoid concurrency issues you must specify both the current number and the number you want to change to.
If current number is not equal to the specified current number the new number will not be set.
True is returned if the sequence number was updated.

Parameter     | Description
------------- | ---------------------
currentNumber | The caller's view of what the current number is
newNumber     | The number to set. The next call to *nextNumber()* will get this number
name          | Name of domain class to set number for
group         | Optional sub-sequence if multiple sequence counters exists for the domain class
tenant        | Tenant ID in a multi-tenant environment

**Long refresh(Class clazz, String group = null, Long tenant = null)**

Sequences are kept in memory for performance reasons and periodically written to the database.
Calling refresh() will discard any updates made in memory and re-initialize the sequence from database.

**void shutdown()**

Flush all sequences and terminate the service.

**List<Map> statistics(Long tenant = null)**

Return statistics for all sequences defined in the application.
Statistics are returned as a List of Maps with the following keys:

Key    | Value
------ | -----------------
name   | Name of domain class
format | The sequence number format
number | Next number that will be returned for this sequence


## Road Map

### Admin UI
Provide a user interface for managing sequence definitions.
Administrators must be able to change number format and next available number.


## Miscellaneous

- The [GR8 CRM ecosystem](http://gr8crm.github.io) uses sequence-generator plugin to generate customer, order and invoice numbers.
