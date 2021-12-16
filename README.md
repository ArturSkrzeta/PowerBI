# PowerBI
+ power query memorizes steps - it's like sellf-documenting technology
+ implicit measure - powerbi callculates on its own
+ explicit measure - we create it witn new measure - we are in control and can be reused
fir+ row context - knowledge of the current row or each and every row
+ filter context
+ measure is always calculated dynamically on the fly
+measure is not a really part of the table - there is no row context - measures are always calculated dynamically and are not stored in the file - what power bi keeps it's only measure's definitio - more efficient when doing  mat - when creating a measure wee need to wrapp column names up into aggregation function as it doesn't have that row context - needs to take entire column but on the dashboard it still can be drilled down or filtered or brokend down with higher grainuality
+ measures should be used whenever numeric calculation needed
+ measures can be reused and shared between other dimensions
+calculated column has  a row context and it's part of a table (storead back in the file which increases its size)
+ compsite/hybrid measure - calculation based on other measures
+ powerbi is based on tabular technology in memory - model compressed and kept in computer's emory
+ in query editor we define a parameter which can be access from power bi level - here we can click 'edit querie' and then 'edit parameters'
+ parameter being set in query editor can be assigned to a field's filter equals to paramter's value
+ parameters cannot be edited from powerbi.com 
+ filters dont work for grouping by when it's done with query editor or dax - - the best way is to drag and drop fields to col/rows/values

+ applying calculation to a filter context 
tuple: value for a set of columns
filter: table of tuples
filter context: set of filters

filter context:
+ it's being made by anything in rows/columns/slicers/filters/filters applied in the calculation
+ rows outside of the filter context are not considered for the computation
+ filter context automatically propagates along the relationship - we transfer filter from one to many
+ filter context by default is empty - if you have no filters you have an access to the entire data model - all rows of the table

row contex:
+ exists only for calculated column or for iterators
+ whenever we write an expression in calclated column it's being created automatically
+ current row for which calulcation is being made
+ can be intorduced with an iterator - sumx or a measure is executed in the row context anyway
+ allows to perform operations on the table one row at a time - iterating over a table
+ when row context is set up - the active relationship is being deactivated (along with filters) for the time of operation execution
+ iterates rows and row context is valid for the rows active in the current filter context
+ doesn't propagates through the relationship

evaluation context:
- filter context + row context

SUMX (
  Sales,
  Sales[...] * Sales[...]
)

+ first argument is a table reference - but it may not retrieve all the row of the table - it retrieves all the row taht are visible trough the current filter context
+ in the second argument the expression is executed in the row contexst -  at that point filter context doesn't matter becuase I don' hva eany aggregations
+ filter context affects aggregat. not column reference 
+ x functions itroduces row context within a  filter context set by a view


expanded table 
- each base table is expanded with the columns of related tables
- whenever we put a filter in the filter context, we always reference the expanded table not tbe base on

DAX
- the way to manipulate the filter context
- with DAX we can remove a filter from the filter context - function ALL() ignores filter context but CALCULATE() can manipulate it
- data analysis expression
- data model: lookup tables provide supporting information to the main table - they are as foreign keys in the main table
- measure is a calculation accross a data model
- with quick measureas we can build dax formulas as with wizard help
- with additional column we can apply a slicer to it or filters to it - when creating a measure then there is no suach a possibility to create a slicer against it
- iterator function: SUMX - return a sum of expression evaluated for each row
- Dax can modify the filter context before executing actual calculation
- when we want to achieve good performance with DAX. we have to manipulate the filter context  and then execute the calculat

CALCULATE (
  SUMX (
    Sales,
    Sales[...] * Sales[...]
  ),
  Product[Color] IN {"Red", "Blue", "Green"}      - - - - filter as table with 3 rows
}

SUMX alone respects the filter context - only calculate() can override the filter context

CALCULATE is an only function that change and override a filter context. it can apply a filter to an expression.
CALCULATE )
<expression>,
<filter1>,
<filter2>
)
+ expression is an aggregate function or table expression itself
+ Calculate evaluates expression in a context modified by filters
+ calculate is being read from bottom to up - first applying filtr then evaulating expression - similar to sql select first from where then select columns

ALL() 
returns all the rows in the table or all the values in the column ignoring any filters tat might have been applied - ignoring currentt context filter and performing calculation on enitre column/table

DIVIDE (
  [Total Sales].
  CALCULATE (
    [Total Sales].
    ALL('Regions'[Country])))

firts total sales respects context filter
dividing by total sales that ignores the filter

+ outer filter context - outside of calculate
+ inner filter context - result of outer filter context modified by the second argument of CALCULATE
+ a filter is a table

CALCULATE and ALL takes off context filter from the calculation

CALCULATE (
  [Total Sales],
  ALL('Regions'[Countires]))

CALCULATE (
  [Total Sales],
  FILTER (
    ALL('Regions'[Countires])),
    'Regions'[Countries] = "Poland"
  )
)

+ with filter I generate a table expression - in fact a table with once column and 1 row - one more filter being appended to a filter contex
+ ALL can be used a table functio or can be used as a filter removal
+ REMOVEFILTER() remover filtera as ALL() does
+ applying a new filter against the same column on which there is a filter in the filter context already, it overrides the old filter
+ KEEPFILTERS() keeps old filters in the filter context whenever whe apply a new filter against the same colum

Quick calcs:
+ gets filtered along with context - it refers to what is currently in the view
+ dax calculation overrides context filter - it refers to all records regardless of what is in the view

iterator filter:
Filter (
  <Table or table expression>,
  <Filter expression>)
- first argument is what iterator iterates through
- filter returns table expression tha has been filtered


SUMX (
  FILTER (
    Orders,
    Orders[Price] > 1
  ),
  Orders[Quantity] * Orders[Price]
)

+filter within sumx is another iterator - it iterates all records visible in the current filter context - for each row evaluates price > 1

naigation function:
+ vlookup type function: taking values from another table besed on the realtion
+ RELATEDTABLE()
+ activating filters from relationship

MAXX:
- x means it iterates row by row
- result is a single column


table expression

context transition:
+ every time we calculate in a row context - the row context becomes a filter context
+ we filtering the filter context by the filter of a current row

+ with a measure we cannot slice a dashboard, we can do this with calculated column
+ disconnected table - a table without any relationship in the data model

Related:
+ retrieving one value
+ works like lookup function - searches by relationship and revtrieves back corresponding value
Related table:
+ retrieving table 

modelowanie danych:
+ zatkanie narzedzia analitcznego baza tranzakcyjna to jest zaden problem
+ ubranie danych w taki schemat tabe,l zeby mozliwe bylo raportowanie wszystkich miar wedlug wszystkich wymiarow jakie sobie zalozymy
+ schemat gwiazd: w centrum tabela faktow, dookola tabele z wymiarami - tabela faktow - tylko miary i klucze obce do wymiarow
+ schemat snowfla: w centrum tabela faktow, dookola tabele z wymiaram, ktore rownize maja tabele wymiarow
+ normalizacja: proces gwarantujacy wspolbizenosc tranzakcyjnej bazy danych - w trakcie edycji danych w jeden tabeli, zeby pozostale nie byly blokow - rozkladanie danych pomiedzy tabele - jedna tablea opisuje jeden obiekt biznesowy (jaki mamy w swiecie,a ktory sprowadzamy do systemu bazy danych)
+ cecha systemow tranzakcyjnych jest ciagla edycja danyc
+ denormalizacja - tabele laczymy w wieksze calos - sporwadzenie platku sniegu do schmatu gwiazdy
+ 3 schmat: wszystkie dane w jednej tabeli - nie trzeba tworzyc relacji do innych tabel - wsad do naczania maszynowego
+ gwiazdozbior -wiele tabel faktow - nie ma zadnej klacznosci pomiedzy gwiazdami
+ hyperspace - gwiazdy polaczon - tabele faktow polaczone wspolnym wymiarem (tabela kalendarza na przyklad)
+ gwiazdy i platki sniegu polaczone ze soba wymiarami lub bezposrednio do faktow
+ klucz obcy to nie jeste relacja, a ograniczenie (constraint) - wartosci w kolumnie sa ograniczone tylko do wartosci 

+ star schema : transaction table that is sourrounded by descriptive tables of dimensions that describe the data
+ fact table alwyas many side and dimension table always one side when it comes to one to may relationship
