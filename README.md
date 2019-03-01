### Trigger-Parking-Lot
Staging place until the applications can be sorted.

##### Create related contact when Account is created

trigger accountAfter on Account (after insert) {
   List<Contact> cons=new List<Contact>();  
    for(Account a: Trigger.New){
        Contact c=new Contact();
        c.accountid=a.id;
        c.lastname=a.name;
        c.phone=a.phone;
        cons.add(c);
    }
    insert cons;
}



##### Prefix first name with Dr when new Lead is created or updated

public class PrefixDocotor on Lead(before insert, before update)
{
	for(Lead l:trigger.new)
	{
		l.FirstName ='Dr' + l.FirstName;
	}
}


@isTest
public class PrefixDoctorTest {
    Public static testMethod void PreDoctor(){
        Lead l=new Lead();
        l.FirstName='Dr Swati';
        l.company='sfdcAmplified';
        l.Status='Open - Not Contacted;
		insert l;
		Lead le = [Select id,FirstName from Lead where company='sfdcAmplified'];
		System.assertEquals(le.FirstName.ContainsIgnoreCase('Dr'),'Dr');
    }
}


##### Update Account Rating to 'Hot 'on account when opportunity stage equals 'closed one'

trigger updateAccountRating on opportunity(after insert,after update)
{
    Set<id> Accountids = new set<id>();
    List<Account> Accounts = new List<Account>();
        if(trigger.new != null)
        {
            for(opportunity opp:trigger.new)
            {
                if(opp.StageName =='Closed Won')
                    {
                  	  AccountIds.add(opp.AccountId);
                    }
            }
        }
    List<Account> a  = [Select id,Rating from Account where Id IN: AccountIds];
    if(a != null){
        for(Account acc: a)
       {
          acc.Rating ='Hot';
          Accounts.add(acc);
       }
    }
          update Accounts;

//test class for the trigger
@istest
public class testClassForTrigger{
	public void testmethod triggerTest{
	//link account with opportunity
		account acc = new account();
		acc.name='test';
		insert acc;
		
		account acc1 = new account();
		acc1.name='test1';
		insert acc1;
		
		opportunity opp = new opportunity();
		opp.name='test opp';
		opp.stagename='closed won';
		opp.AccountId = acc.id;
		insert opp;
		
		opportunity opp1 = new opportunity();
		opp1.name='test opp';
		opp1.stagename='closed won';
		opp1.AccountId = acc1.id;
		insert opp1;


		test.startTest();  
		List<Account> acc = [Select id,rating from account where id =: opp.AccountId];
		acc.get[0]; //return acc object
		system.assertequals(acc.get[0].Rating,'hot');
		test.stopTest();
	}
}
}

##### Whenever phone field is updated in account then the name field should also get updated with name and phone number in accounts

Trigger updateName on Account (before update){	

for(account acc:trigger.new)
		{
			if(acc.phone != trigger.oldMap.get(acc.id).phone)
			{			
				acc.name = acc.name + acc.phone
			}
		}
}

##### Prevent account from deleting, if it has 2 or more contacts

trigger deleteacc on Account (before delete) {
Set<id> Accountids = new Set<id>();
		for(account acc:trigger.old)
		{
		Accountids.add(acc.id);
		}
	
List<Account> account = [select id,name,(select id, name from contacts) from account where id in: Accountids];
Map<id,Account> mapacc = new Map<id,Account>();
for(account a:account){
    mapacc.put(a.id,a);
} 
    //for showing error use context variable bcoz error shows on context variable
    for(account acc:trigger.old)
		{
		if(mapacc.get(acc.id).contacts.size()>=2)
				{
				acc.adderror('account cannot be deleted');
				}
		}
	}
  
  ##### When lead is created or updated then check if the email of lead is already there in existing contacts. If email already exist then throw error.
  
  trigger DuplicateEmailsInLead on Lead (before insert, before update) {
    map<String,Contact>  mapOfContact = new map<string,Contact>();
    list<contact> con = [select id,email from contact];
     for (contact c:con)
     {
        mapofcontact.put(c.email,c);
   	 }
        for(lead l : trigger.new)
        {
                    if((l.email != null) && (trigger.isInsert || (l.email != trigger.oldmap.get(l.id).email))){
                        if(mapofContact.containsKey(l.email)){
                            l.Email.addError('Email already exists');
                  }                  
           }               
     }
}
  
  ##### Write a trigger to update a field (city) in opportunity when same field(city) is updated in account
  
trigger AccountUpdate on Account (after update) {
Set<id> AccountIds = new Set<id>();
    for(Account a: trigger.new)
    {
        if(a.city__c != trigger.oldmap.get(a.id).city__c){
        AccountIds.add(a.id);
    }
    }
    List<opportunity> opport=[select id, City__c,Accountid from Opportunity where accountid in: Accountids];
    
        for(opportunity opp:opport)
        {
                     opp.City__c = trigger.newmap.get(opp.accountid).city__c;
        }
}

##### Whenever TestPhoneOpportunity__c field in opportunity is updated ,its related field (TestPhoneAccount__c) in Account and (TestPhoneContact__c ) in Contact should get updated with the updated value with TestPhoneOpportunity__c

trigger oppupdate on Opportunity(after update);
{
Set<id> accountIds  = set <id>();
List<contact> cc = new List<contact>();
	if(trigger.new!=null)
	{
		for(opportunity opp:trigger.new)
        if(opp.TestPhoneOpportunity__c != trigger.oldmap.get(opp.id).TestPhoneOpportunity__c){
			

			accountIds.add(opp.AccountId);
		}
	}
	Map<String,opportunity> accountidtoopp = map<String,opportunity>();
	List<opportunity> opp=[Select id,TestPhoneOpportunity__c,AccountId from opportunity where Accountid IN:accountids];
	for (opportunity op:opp)
	{
	accountidtoopp.put(op.accountid,op);
	}	
	
	List<Account> acc = [Select id,TestPhoneAccount__c, (Select id,TestPhoneContact__c,AccountId from contacts )from Account where Id IN: accountIds ];
	
	for(Account a:acc)
	{
		a.TestPhoneAccount__c = accountidtoopp.get(a.Id).TestPhoneOpportunity__c;
		for(contact c: a.contacts){
		c.TestPhoneContact__c = accountidtoopp.get(c.AccountId).TestPhoneOpportunity__c;
		cc.add(c);
		}
	}
	update acc;
	update cc;
	
  
  ##### When an opportunity is inserted or updated then if the stage name is 'Closed won' then add the task.
  
  trigger ClosedOpportunityTrigger on Opportunity (after insert, after update) {


   List<Task> taskList = new List<Task>();
    for (Opportunity o :[SELECT Id,StageName FROM Opportunity WHERE StageName ='Closed Won' AND Id IN :Trigger.New]) 
    {
        if(o.StageName == 'Closed Won')    
        {
            taskList.add(new task (Subject ='Follow Up Test Task' , WhatId=o.Id));
        }
            
    }
    if(taskList.size() > 0){
         insert taskList;
   }
}

##### Whenever new account is created with annual revenue more than 50,000 then add 'John Doe' as contact name

Trigger annualRevenueAcc on Account(after insert){
List<Contact> cc  = new list<Contact>();


      for(Account acc:trigger.new)
    {
            if(acc.AnnualRevenue > 50,000)
              { 
			  contact con = new contact();
                 con.FirstName ='Smriti';
                 con.lastName ='Sharan';
				 con.accountid=acc.id;
				 cc.add(con);
              }  
    }      
	          insert cc;

                      
         }
         
         
  #####  Whenever case is created with origin as email then set status as new and priority as Normal
  
  Trigger emailCase on Case(before insert){
         for(case c:trigger.new)
  {
            if(c.origin =='Email')
              {
                 c.status ='New';
                 c.priority ='Normal';
              }
  }
}

##### Whenever account phone is modified then update contact record with phone field (otherphone with oldvalue and homephone with new value) associated with account records.

Trigger phoneAcc on Account(after update)
{
List<Contact> cc = new List<Contact>();
Set<id> accid = new Set<id>();
	{
       for(Account acc:trigger.new)
	   {
            if(acc.phone != trigger.oldMap.get(acc.id).phone){
	        accid.add(acc.id); 
             }
	   }
	}
if(!accId.isEmpty){
List<Account> ac= [Select id,name,(Select id, otherphone,homephone from contacts) from Account where Id =: accid];
}

for(Account acc:ac)
	{
	  
        for(contact c: acc.contacts){
           c.homephone = trigger.newMap.get(acc.id).phone;
		   c.otherphone = trigger.oldMap.get(acc.id).phone;
           cc.add(c);
          }    
     		
	}
  if(!cc.isEmpty()){
		update cc;
      }
}


##### When a new contact is created for existing account then set contact other phone as account phone

Trigger  phoneCon on Contact(before insert){
	  
	  Set<id> accIds = new Set<id>();
	  for(Contact c:trigger.new){
	  accIds.add(c.AccountId)
	  }
	  
	  Map<id,Account> accmap = new Map<id,Account>();
	  List<Account> a = [select id,phone from Account where id IN:accIds ];
	  for(account acc:a)
	  {
	     accmap.put(a.id,a)
	  }
	  
	  for(Contact con:trigger.new)
	  {    
	      if(con.accountId =! null)
		  {
		    con.otherphone =accmap.get(con.accountId).phone;
		  }
	  }
    }
    
##### When a new Account record is inserted verify the industry field value, if industry field value is Education then assign the owner as John Doe

Trigger industryAcc on Account(before insert){
User u=[select id from User where username='smriti@sfdc.com'];

     for(Account acc:trigger.new)  
 {     

    if(acc.Industry =='Education')      
   {          
      acc.ownerId = u.id;    
  
   } 

 }
}

##### The following Trigger will fires when we try to create the account with same name i.e. Preventing the users to create Duplicate Accounts

trigger DuplicateAccount on Account (before insert) {
List<String> duplicateName = new List<String>();
Set<String> aName = new Set<String>();
List <Account> listAcc = new List<Account>();


for(Account acc:trigger.new)
{
   aName.add(acc.Name);
}
//duplicate records of account in the list 
 List<Account> acc = [Select id,Name from Account where Name IN: aName];
	 //putting duplicate in list
	 for(Account ac:acc)
	{
	   duplicateName.add(acc.Name);
	}


//iterating over all account and seggregate list which contains duplicate name 
	for(Account ac: trigger.new)
	{
	   if(duplicateName.contains(ac.name)){
			  listAcc.add(ac);
	   }
	}
	for(Account acc:listAcc)
	{
		  acc.AddError('You cannot create account with the same name');
	}
 }
 
 
