---
layout: default
title: Library
---

## Library
Author: Peter Gorm Larsen and Alvaro Miyazawa and Jim Woodcock


This is the library example which originally was posed in the early 1980's and a study
of 12 specifications of the problem was examined in: "A Study of 12 Specifications of 
the Library Problem" by Jeanette Wing, July 1988 in IEEE Software. In this model a VDM
part was introduced by Peter Gorm Larsen and the reactive part was added by Alvaro 
Miyazawa and Jim Woodcock. The core of this model illustrate the use of most of the
CML data types. The way guards in the reactive parts are made illustrate how to reach 
deadlock situations using the CML debugger.


### Library.cml

{% raw %}
~~~
types
  Command = Borrow | Renew | Return | Find
  
  String = seq of char
  BookId = token
  UserId = token
  BorrowMap = map UserId to set of BookId
  
  Book ::  
    title : String
    author : String 

  User :: 
      name : String 
      
  Borrow ::
    copy : BookId
    user : UserId
   
  Renew ::
    copy : BookId
    user : UserId
  Return :: copy : BookId
  Find :: string : String
  
  Library ::
    books : map BookId to Book
    users : map UserId to User
    borrowed : BorrowMap
  inv mk_Library(bs,us,bor) ==
    dom bor subset dom us
    and dunion rng bor subset dom bs
    and (forall u1, u2 in set dom bor @ u1 <> u2 => bor(u1) inter bor(u2) = {})
    
  values
  user1 = mk_User("User1")
  user2 = mk_User("User2")
  book1 = mk_Book("Book1", "Author1")
  book2 = mk_Book("Book2", "Author2")
  users = {mk_token(user1) |-> user1, mk_token(user2) |-> user2}
  books = {mk_token(book1) |-> book1, mk_token(book2) |-> book2}
  
  functions
  ExeBorrow: Borrow * Library -> Library
  ExeBorrow(b,l) ==
    mk_Library(l.books,
               l.users,
               l.borrowed++{b.user |-> BorrowCopy(b.user,b.copy,l.borrowed)})
  pre
    b.copy in set dom l.books
    and b.user in set dom l.users
    and b.copy not in set dunion rng l.borrowed
    
  BorrowCopy: UserId * BookId * BorrowMap -> (set of BookId)
  BorrowCopy(u,c,bor) ==
    if u in set dom bor
    then bor(u) union {c}
    else {c}
    
  ExeReturn: Return * Library -> Library
  ExeReturn(r, l) ==
    mk_Library(l.books,l.users,RemoveCopy(r.copy,l.borrowed))
  pre r.copy in set dom l.books

  RemoveCopy: BookId * BorrowMap -> BorrowMap
  RemoveCopy(c,bor) ==
    {u |-> bor(u) \ {c} | u in set dom bor}
    
  ExeFind: String * Library -> (set of BookId)
  ExeFind(s,l) ==
    {bid | bid in set dom l.books @ s in set {l.books(bid).title, l.books(bid).author}}
  pre card({bid | bid in set dom l.books @ 
                  s in set {l.books(bid).title, l.books(bid).author}}) > 0
    
  channels
  borrow: Borrow
  retBook: Return
  renew: Renew	
  loans: UserId*nat
  find:Find
  
  process LibraryProcess = begin
  state 
    library: Library := mk_Library({|->},{|->},{|->})
    booksP : set of BookId := {}
  inv
    (dom(library.borrowed) = dom(library.users)) and
    (dunion rng(library.borrowed) subset dom(library.books))

    
  functions	
    GetBorrowed: Library -> BorrowMap
    GetBorrowed(l) == l.borrowed
		
    ApplyBorrowed: Library*UserId -> set of BookId
    ApplyBorrowed(l,u) == (GetBorrowed(l))(u)
    
    operations
    Init: map BookId to Book * map UserId to User ==> ()
    Init(bs, us) ==
      (library := mk_Library(bs,us,{u|->{}| u in set dom us});
       booksP := {})
        
    actions
    Act = 
      ( 
        borrow?b -> [pre_ExeBorrow(b,library)] & (library := ExeBorrow(b,library))
        []
        renew?r -> [pre_ExeReturn(mk_Return(r.copy),library)] & 
          (library := ExeReturn(mk_Return(r.copy),library);
           library := ExeBorrow(mk_Borrow(r.copy,r.user),library))
        []
        retBook?r1 -> [pre_ExeReturn(mk_Return(r1.copy),library)] & (library := ExeReturn(mk_Return(r1.copy),library))
        []
        find?f -> (booksP := ExeFind(f.string,library))
        []
        loans?u!(card(ApplyBorrowed(library,u))) -> [ (u in set dom(GetBorrowed(library))) ] & Skip
      ); Act
	  
  @
    Init(books,users); Act
end
~~~
{% endraw %}

