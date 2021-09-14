import pandas as pd

df = pd.read_csv('purchase_data.csv')

print("===============================================")
print("PLAYER COUNT:")
print("Total Number of Players:", len(list(df['SN'].unique())) )
print("Purchasing Analysis (Total):", sum(df['Price']) )
print("Number of Unique Items:", len(list(df['Item ID'].unique())) ) 
print("Total Number of Purchases:", df['Purchase ID'].count()) 
print("Average Purchase Price:", sum(df['Price'])/df['Purchase ID'].count())
print("Total Revenue:",sum(df['Price']) ) 

print("===============================================")
print("Gender Demographics:")

unique_sn_df=df.drop_duplicates(subset=['SN'])

new_gender_df=unique_sn_df.groupby('Gender', as_index=False).size()


for index, rows in new_gender_df.iterrows():
  print(f"Percentage and Count of {rows['Gender']} Players:")
  print("Count:",rows['size'])
  print("Percentage:", rows['size']/new_gender_df['size'].sum()*100 )

print("===============================================")

print("Purchasing Analysis (Gender):")

new_gender_purchasing_df=df.groupby('Gender', as_index=False).size()


for index, rows in new_gender_purchasing_df.iterrows():
  print(f"Purchasing Analysis ({rows['Gender']}):")

  print("Purchase Count:", df[df['Gender']==rows['Gender']]['Purchase ID'].count())
  print("Total Purchase Value:", df[df['Gender']==rows['Gender']]['Price'].sum())
  print("Average Purchase Price:", df[df['Gender']==rows['Gender']]['Price'].sum()/new_gender_df[new_gender_df['Gender']==rows['Gender']]['size'].values[0])

  unique_sn_gender_df=unique_sn_df[unique_sn_df['Gender']==rows['Gender']]['SN'].to_frame() 
  avg_purchase_df=pd.DataFrame(columns = ['SN', 'Gender', 'Average Purchase Total'])
  for i_index, i_rows in unique_sn_gender_df.iterrows():
    avg_purchase_df = avg_purchase_df.append({'SN' :i_rows['SN'] , 'Gender' : rows['Gender'], 'Average Purchase Total' : df[df['SN']==i_rows['SN']]['Price'].sum()/new_gender_df[new_gender_df['Gender']==rows['Gender']]['size'].values[0]}, ignore_index = True)
  print("Average Purchase Total per Person by Gender:")
  print(avg_purchase_df)

print("===============================================")

new_age_df=unique_sn_df.groupby('Age', as_index=False).size()

print("Age Demographics:")

print("Age Demographics: age<10")
print("Purchase Count:",df[df['Age']<10]['Purchase ID'].count())
print("Total Purchase Value:", df[df['Age']<10]['Price'].sum())
print("Average Purchase Price:", df[df['Age']<10]['Price'].sum()/len(unique_sn_df[unique_sn_df['Age']<10]))

unique_sn_age_df=unique_sn_df[unique_sn_df['Age']<10][['SN','Age']] 
avg_purchase_byage_df=pd.DataFrame(columns = ['SN', 'Age', 'Average Purchase Total'])
for i_index, i_rows in unique_sn_age_df.iterrows():
  avg_purchase_byage_df = avg_purchase_byage_df.append({'SN' :i_rows['SN'] , 'Age' : i_rows['Age'], 'Average Purchase Total' : df[df['SN']==i_rows['SN']]['Price'].sum()/new_age_df[new_age_df['Age']==i_rows['Age']]['size'].values[0]}, ignore_index = True)
print(f"Average Purchase Total per Person by Age Group: age<10")
print(avg_purchase_byage_df)

avg_purchase_byage_df=pd.DataFrame()

for agenum in range(10,100,5):
  print(f"Age Demographics: age: {agenum}-{agenum+4}")
  if df[(df['Age']>=agenum) & (df['Age']<=(agenum+4))]['Purchase ID'].count()>0:
    print("Purchase Count:",df[(df['Age']>=agenum) & (df['Age']<=(agenum+4))]['Purchase ID'].count())
    print("Total Purchase Value:", df[(df['Age']>=agenum) & (df['Age']<=(agenum+4))]['Price'].sum())  
    print("Average Purchase Price:", df[(df['Age']>=agenum) & (df['Age']<=(agenum+4))]['Price'].sum()/len(unique_sn_df[(unique_sn_df['Age']>=agenum) & (unique_sn_df['Age']<=(agenum+4))]))

    
    unique_sn_age_df=unique_sn_df[(unique_sn_df['Age']>=agenum) & (unique_sn_df['Age']<=(agenum+4))][['SN','Age']] 
    avg_purchase_byage_df=pd.DataFrame(columns = ['SN', 'Age', 'Average Purchase Total'])
    for i_index, i_rows in unique_sn_age_df.iterrows():
      avg_purchase_byage_df = avg_purchase_byage_df.append({'SN' :i_rows['SN'] , 'Age' : i_rows['Age'], 'Average Purchase Total' : df[df['SN']==i_rows['SN']]['Price'].sum()/new_age_df[new_age_df['Age']==i_rows['Age']]['size'].values[0]}, ignore_index = True)
    print(f"Average Purchase Total per Person by Age Group:{agenum}-{agenum+4}")
    print(avg_purchase_byage_df)

  else:
    print("No records")


print("===============================================")
print("Top Spenders:")


top_spender=df.groupby(['SN'], as_index=False).sum().sort_values('Price', ascending=False).head(5)

top_spender_df=pd.DataFrame(columns = ['SN', 'Purchase Count', 'Average Purchase Price','Total Purchase Value'])

for index,rows in top_spender.iterrows():
  top_spender_df = top_spender_df.append({'SN' : rows['SN'], 'Purchase Count' : df[df['SN']==rows['SN']]['Purchase ID'].count(), 'Average Purchase Price' : rows['Price']/df['Purchase ID'].count(), 'Total Purchase Value' : rows['Price']}, ignore_index = True)

print(top_spender_df)

print("===============================================")
print("Most Popular Items:")

most_popular_item=df.groupby(['Item ID','Item Name'], as_index=False).size().sort_values('size', ascending=False).head(5)

most_popular_item_df=pd.DataFrame(columns = ['Item ID', 'Item Name', 'Purchase Count','Item Price','Total Purchase Value'])

for index,rows in most_popular_item.iterrows():
  get_highest_occurance_price=df[df['Item ID']==rows['Item ID']][['Item ID','Price']].groupby(['Item ID','Price'], as_index=False).size()['size'].nlargest(1).values[0]

  get_occurance_price_df=df[df['Item ID']==rows['Item ID']][['Item ID','Price']].groupby(['Item ID','Price'], as_index=False).size() 

  most_popular_item_df = most_popular_item_df.append({'Item ID' : rows['Item ID'], 'Item Name' :rows['Item Name'], 'Purchase Count' : rows['size'],'Item Price' : get_occurance_price_df[get_occurance_price_df['size']==get_highest_occurance_price]['Price'].values[0] ,'Total Purchase Value' : df[df['Item ID']==rows['Item ID']]['Price'].sum()}, ignore_index = True)
print(most_popular_item_df)

print("===============================================")
print("Most Profitable Items:")


most_profitable_item=df.groupby(['Item ID','Item Name'], as_index=False).sum().sort_values('Price', ascending=False).head(5)

most_profitable_item_df=pd.DataFrame(columns = ['Item ID', 'Item Name', 'Purchase Count','Item Price','Total Purchase Value'])

for index,rows in most_profitable_item.iterrows():
  get_highest_occurance_price=df[df['Item ID']==rows['Item ID']][['Item ID','Price']].groupby(['Item ID','Price'], as_index=False).size()['size'].nlargest(1).values[0]

  get_occurance_price_df=df[df['Item ID']==rows['Item ID']][['Item ID','Price']].groupby(['Item ID','Price'], as_index=False).size()

  most_profitable_item_df = most_profitable_item_df.append({'Item ID' : rows['Item ID'], 'Item Name' :rows['Item Name'], 'Purchase Count' : 0,'Item Price' : get_occurance_price_df[get_occurance_price_df['size']==get_highest_occurance_price]['Price'].values[0] ,'Total Purchase Value' : rows['Price']}, ignore_index = True)

print(most_profitable_item_df)
