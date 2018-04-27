#include<bits/stdc++.h>
#include<ctime>
using namespace std;

int binary_tree(int node_row,int node_colm,int index_left,int index_right,int node_left,int node_right,int inside_block,int blk_size,int tree[][100])
{
    int node_middle=(node_right+node_left)/2;
    if(index_right==node_right&&index_left==node_left)
    {
        return tree[inside_block*blk_size+node_row][node_colm];
    }
    else if(index_left<=node_middle)
    {
        if(index_right<=node_middle)
        {
            return binary_tree(2*node_row,node_colm-1,index_left,index_right,node_left,node_middle,inside_block,blk_size,tree);
        }
        else
        {
            int m=binary_tree(2*node_row,node_colm-1,index_left,node_middle,node_left,node_middle,inside_block,blk_size,tree);
            int n=binary_tree(2*node_row+1,node_colm-1,node_middle+1,index_right,node_middle+1,node_right,inside_block,blk_size,tree);
            if(m>n)
            {
                return n;
            }
            else
            {
                return m;
            }
        }
    }
    else
    {
        return binary_tree(2*node_row+1,node_colm-1,index_left,index_right,node_middle+1,node_right,inside_block,blk_size,tree);
    }
}




int main()
{
    float start_time;                 //start_time is the start time of the execution of the program
    float preprocessing_time;         //preprocessing_time is the time taken in the preprocessing phase
    float query_time;                 //query_time is the time of a single query
    start_time=clock();
    int len1;                           //len1=length of the input array
    cin >> len1;        
    int blk_size=log2(len1)/2;          //blk_size is the size of 1 block
    int len=len1%blk_size;              
    if(len!=0)
    {
        len=blk_size-len+len1;    
    }
    else
    {
        len=len1;
    }
    int arr[len];                       //arr[] is the array containing the input sequence
    int x;
    for(int i=0;i<len1;i++)
    {
        cin >> x;
        arr[i]=x;
    }
    for(int i=0;i<len-len1;i++)
    {
        arr[i+len1]=65500;
    }



    int forward[len];                   //forward[] is the array that stores the minimum of the block starting from any location "i"
    int backward[len];                  //backward[] is the array that stores the minimum of the block ending at any location "i"
    int count=0;
    int min=0;
    for(int i=0;i<len;i++)
    {
        if(blk_size!=1)
        {
            if(count==0)
            {
                min=arr[i];
                count++;
            }
            else if(count==blk_size-1)
            {
                count=0;
                if(min>arr[i])
                {
                    min=arr[i];
                }

            }
            else
            {
                if(min>arr[i])
                {
                    min=arr[i];
                }
                count++;
            }
            backward[i]=min;
        }
        else
            backward[i]=arr[i];
        
    }
    
    
    min=0;
    count=0;
    for(int i=len-1;i>-1;i--)
    {
        if(blk_size!=1)
            {
            if(count==0)
            {
                min=arr[i];
                count++;
            }
            else if(count==blk_size-1)
            {
                count=0;
                if(min>arr[i])
                {
                    min=arr[i];
                }

            }
            else
            {
                if(min>arr[i])
                {
                    min=arr[i];
                }
                count++;
            }
            forward[i]=min;
        }
        else
            forward[i]=arr[i];
    }
    int num_blks=len/blk_size;              //num_blks is the number of blocks
    int num_colm=log2(num_blks)+1;          //num_colm is the number of columns in the sparse table
    int sparse_table[num_blks][num_colm];   //sparse_table[][] contains the entries which are present in sparse table data-structure
    for(int i=0;i<num_blks;i++)
    {
        sparse_table[i][0]=forward[i*blk_size];
    }
    int y=1;
    int i=2;
    while (i < num_blks+1)                  //populating the sparse_table[][]
    {
        for (int j=0; j<num_blks-i+1; j++)
        {
           if(sparse_table[j][y-1] <= sparse_table[j+int(pow(2,y-1))][y-1])
               sparse_table[j][y] = sparse_table[j][y-1];
           else
              sparse_table[j][y]=sparse_table[j+int(pow(2,y-1))][y-1];
        }
        i = i*2;
        y = y+1;
    }

    
    
    int tree[len][100];                 //tree[][] is a variant of segment trees data structure 
    
    for(int i=0;i<len;i++)
    {
        tree[i][0]=arr[i];
    }
    int c=0;                            
    if(pow(2,log2(blk_size))-blk_size==0)
        c=log2(blk_size)+1;
    else
        c=log2(blk_size)+2;             //c is the height of the tree structure
    int n=blk_size;
    int p=1;
    for(int j=0;j<num_blks;j++)
    {    
        n=blk_size;
        p=1;
        while(n!=1)
        {
            for(int i=0;i<(n/2);i++)
            {
                if(tree[j*blk_size+2*i][p-1]>tree[j*blk_size+2*i+1][p-1])
                    tree[j*blk_size+i][p]=tree[j*blk_size+2*i+1][p-1];

                else
                    tree[j*blk_size+i][p]=tree[j*blk_size+2*i][p-1];
            }
            if(n%2==1)
            {
                tree[j*blk_size+n/2][p]=tree[j*blk_size+(n/2)*2][p-1];
                n++;
            }
            n=n/2;
            p++;
        }
    }/*
    for(int i=0;i<len;i++)
    {
        for(int j=0;j<c+1;j++)
        {
            cout << tree[i][j] << " ";
        }
        cout << endl;
    }
    cout << endl;
    for(int i=0;i<num_blks;i++)
    {
        for(int j=0;j<num_colm;j++)
        {
            cout << sparse_table[i][j] << " ";
        }
        cout << endl;
    }
    cout << endl;
    for(int i=0;i<len;i++)
    {
        cout << forward[i] << " ";
    }
    cout << endl;
    
    for(int i=0;i<len;i++)
    {
        cout << backward[i] << " ";
    }
    cout << endl;
    */
    
    
    preprocessing_time=(clock()-start_time);
    preprocessing_time=((float)preprocessing_time)/CLOCKS_PER_SEC;
    //cout<<preprocessing_time << " preprocessing "<<endl;
    int num_cases;                  //num_cases is the number of queries the user requests
    cin>>num_cases;
    float avg_time=0;               //avg_time is the average time taken by each query
    for (int i=0; i <num_cases; i++)
    {
        start_time=clock();
        int x, y;                   //x and y are the indices or the range given in the query for which the minimum is to be determined
        cin >> x >> y;
        if(x/blk_size!=y/blk_size)
        {

            int f=forward[x];
            int b=backward[y];
            x=x/blk_size+1;
            y=y/blk_size-1;
            int z = y-x+1;
            int mini=f;
            if(b<mini)
                    mini=b;
            if(z!=0)
            {
                z=int(log2(z));
                int p = sparse_table[x][z];
                int q = sparse_table[int(y-pow(2,z)+1)][z];
                if(p<mini)
                    mini=p;
                if(q<mini)
                    mini=q;
            }
            cout<<mini<<endl;       //mini stores the result of the given query if x and y are in different blocks
        }
        else
        {
            int c1;
            if(int(pow(2,int(log2(blk_size))))-blk_size!=0)
            {
                c1=c+1;
            }
            else
            {
                c1=c;
            }
            int inside_block=x/blk_size+1;
            x=x%blk_size;
            y=y%blk_size;
            cout<<binary_tree(0,c1-1,x,y,0,blk_size-1,inside_block-1,blk_size,tree)<<endl;
        }
        query_time=clock()-start_time;
        query_time=((float)query_time)/CLOCKS_PER_SEC;
        avg_time=avg_time+query_time;
        //cout<<query_time << " query "<<endl;
    }
    avg_time=avg_time/num_cases;
    //cout<<avg_time<<" average "<<endl;
}