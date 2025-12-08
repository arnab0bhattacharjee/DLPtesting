#include <stdio.h>
#include <stdlib.h>
 
struct complex
{
  int real, img;
};
 
int main()
{
  int choice, x, y, z;
  struct complex a, b, c;
 
  while(1)
  {
    printf("Press 1 to add two complex numbers.\n");
    printf("Press 2 to subtract two complex numbers.\n");
    printf("Press 3 to multiply two complex numbers.\n");
    printf("Press 4 to divide two complex numbers.\n");
    printf("Press 5 to exit.\n");
    printf("Enter your choice\n");
    scanf("%d", &choice);
 
    if (choice == 5)
      exit(0);
 
    if (choice >= 1 && choice <= 4)
    {
      printf("Enter a and b where a + ib is the first complex number.");
      printf("\na = ");
      scanf("%d", &a.real);
      printf("b = ");
      scanf("%d", &a.img);
      printf("Enter c and d where c + id is the second complex number.");
      printf("\nc = ");
      scanf("%d", &b.real);
      printf("d = ");
      scanf("%d", &b.img);
    }
    if (choice == 1)
    {
      c.real = a.real + b.real;
      c.img = a.img + b.img;
 
      if (c.img >= 0)
        printf("Sum of the complex numbers = %d + %di", c.real, c.img);
      else
        printf("Sum of the complex numbers = %d %di", c.real, c.img);
    }
    else if (choice == 2)
    {
      c.real = a.real - b.real;
      c.img = a.img - b.img;
 
      if (c.img >= 0)
        printf("Difference of the complex numbers = %d + %di", c.real, c.img);
      else
        printf("Difference of the complex numbers = %d %di", c.real, c.img);
    }
    else if (choice == 3)
    {
      c.real = a.real*b.real - a.img*b.img;
      c.img = a.img*b.real + a.real*b.img;
 
      if (c.img >= 0)
        printf("Multiplication of the complex numbers = %d + %di", c.real, c.img);
      else
        printf("Multiplication of the complex numbers = %d %di", c.real, c.img);
    }
    else if (choice == 4)
    {
      if (b.real == 0 && b.img == 0)
        printf("Division by 0 + 0i isn't allowed.");
      else
      {
        x = a.real*b.real + a.img*b.img;
        y = a.img*b.real - a.real*b.img;
        z = b.real*b.real + b.img*b.img;
 
        if (x%z == 0 && y%z == 0)
        {
          if (y/z >= 0)
            printf("Division of the complex numbers = %d + %di", x/z, y/z);
          else
            printf("Division of the complex numbers = %d %di", x/z, y/z);
        }
        else if (x%z == 0 && y%z != 0)
        {
          if (y/z >= 0)
            printf("Division of two complex numbers = %d + %d/%di", x/z, y, z);
          else
            printf("Division of two complex numbers = %d %d/%di", x/z, y, z);
        }
        else if (x%z != 0 && y%z == 0)
        {
          if (y/z >= 0)
            printf("Division of two complex numbers = %d/%d + %di", x, z, y/z);
          else
            printf("Division of two complex numbers = %d %d/%di", x, z, y/z);
        }
        else
        {
          if (y/z >= 0)
            printf("Division of two complex numbers = %d/%d + %d/%di",x, z, y, z);
          else
            printf("Division of two complex numbers = %d/%d %d/%di", x, z, y, z);
        }
      }
    }
    else
      printf("Invalid choice.");
 
    printf("\nPress any key to enter choice again...\n");
  }
}

/**************       Bplustree.c      *****************/
/*  To Compile      gcc -o  Bplustree  Bplustree.c     */  
/*  To Run          Bplustree  <input                  */
/*******************************************************/
 
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <string.h>

#ifndef _BTREE_

#ifdef __cplusplus
extern "C" {
#endif

int OpenIndex(char *Name, int DupKeys);
/* open or create index file, find free entry in index table,
    fill in entry, DupKeys is 1 if duplicate keys are allowed, 0 if not,
    return number of entry in table */

void CloseIndex(int Index);
/* close Index file, free entry in index info table */

int Insert(int Index, long Key, long Ptr);
/* insert Key & Ptr in index file, return 0 for success, nonzero for error */

long Lookup(int Index, long Key);
/* return pointer for entry in Index file <= Key, return -1 if none */


#ifdef __cplusplus
}
#endif

#endif

#define ORDER 64   
#define BLKSIZE (ORDER*8)
#define BKTPTRS (ORDER*2 - 1)
#define NLEN 40
#define NIDX 20
#define SEEK_SET  0   

typedef struct {
	short LeafNode;
	short KeyCount;
	long Keys[ORDER - 1];
	long Ptrs[ORDER];
} NodeType;

typedef struct {
	long RootBlock;
	long AllocBlock;
	char Junk[BLKSIZE - 2*sizeof(long)];
} HeadType;

typedef struct {
	long Pointers[BKTPTRS];
	long NextBucket;
} BucketType;

typedef struct {
	int Used;
	int File;
	char Name[NLEN];
	int DupKeys;
	long RootBlk;
	long AllocBlk;
	long CurBlk;
	short CurKey;
	short CurPtr;
} IdxInfo;
#define File(Idx)	(IdxTab[Idx].File)
#define Root(Idx)	(IdxTab[Idx].RootBlk)
#define Used(Idx)	(IdxTab[Idx].Used)
#define Dups(Idx)	(IdxTab[Idx].DupKeys)
#define Alloc(Idx)	(IdxTab[Idx].AllocBlk)
#define CBlk(Idx)	(IdxTab[Idx].CurBlk)
#define CKey(Idx)	(IdxTab[Idx].CurKey)
#define CPtr(Idx)	(IdxTab[Idx].CurPtr)

BucketType Bucket;  

static IdxInfo IdxTab[NIDX];

static void ReadBlock(int File, long Block, int Size, void *Addr) {
/*    
printf("ReadBlock(File %d, Block %ld, Size %d, Addr %x)\n",
File, Block, Size, Addr);
*/  

	if(lseek(File, Block * Size, SEEK_SET) == -1 ||
	   read(File, Addr, Size) == -1)
		printf("ReadBlock Error, File %d, Block %d, Size %d\n",
			File, Block, Size);
}

static void WriteBlock(int File, long Block, int Size, void *Addr) {
/* 
printf("WriteBlock(File %d, Block %ld, Size %d, Addr %x)\n",
File, Block, Size, Addr);
*/ 
	if(lseek(File, Block * Size, SEEK_SET) == -1 ||
	   write(File, Addr, Size) == -1)
		printf("WriteBlock Error, File %d, Block %d, Size %d\n",
			File, Block, Size);
}

static void ReadHead(int Index) {
	HeadType HeadBlock;
	ReadBlock(File(Index), 0, BLKSIZE, &HeadBlock);
	Root(Index) = HeadBlock.RootBlock;
	Alloc(Index) = HeadBlock.AllocBlock;
}

static void WriteHead(int Index) {
	HeadType HeadBlock;
        int i;
	HeadBlock.RootBlock = Root(Index);
	HeadBlock.AllocBlock = Alloc(Index);
	WriteBlock(File(Index), 0, BLKSIZE, &HeadBlock);
}

int OpenIndex(char *Name, int DupKeys) {
	int i;
	int j;
	for(i = 0; i < NIDX; i++)
		if(!Used(i)) break;
	Used(i) = 1;
	strcpy(IdxTab[i].Name, Name);
	Dups(i) = DupKeys;
	CBlk(i) = -1;
	if((File(i) = open(Name, O_RDWR)) >= 0) {
		ReadHead(i);
	} else
/*	if((File(i) = creat(Name, S_IREAD|S_IWRITE)) >= 0) {    */ 
        if((File(i) = open(Name, O_RDWR|O_CREAT, 0644)) >= 0) { 
		NodeType Node;
		Root(i) = 1;
		Alloc(i) = 2;
		WriteHead(i);
		Node.LeafNode = 1;
		Node.KeyCount = 0;
	        for(j = 0; j < ORDER-1; j++)
                  { 
                    Node.Ptrs[j] = -1; 
                    Node.Keys[j] = 0; 
                   }  
		Node.Ptrs[ORDER - 1] = -1;
		WriteBlock(File(i), 1, BLKSIZE, &Node);
	} else {
		perror("OpenIndex");
		Used(i) = 0;
		i = -1;
	}
	return i;
}

void CloseIndex(int Index) {
	WriteHead(Index);
	close(File(Index));
	Used(Index) = 0;
}

static int FindKey(NodeType *Node, long Key) {
	int k;
	for(k = 0; k < Node->KeyCount; k++)
		if(Node->Keys[k] > Key) break;
	return k;
}

static void CheckBucket(int Index, NodeType *Node, long *Key, long *Ptr) {
	*Ptr = -1;
	if(CBlk(Index) >= 0) {
		long NextPtr;
	*Key = Node->Keys[CKey(Index)-1];
	*Ptr = Node->Ptrs[CKey(Index)];
	if(Dups(Index)) {
/*		BucketType Bucket;   */ 
		ReadBlock(File(Index), *Ptr, BLKSIZE, &Bucket);
		*Ptr = Bucket.Pointers[CPtr(Index)];
		CPtr(Index)++;
		NextPtr = Bucket.Pointers[CPtr(Index)];
	}
	if(!Dups(Index) || NextPtr < 0) {
		CPtr(Index) = 0;
		CKey(Index)++;
		if(CKey(Index) > Node->KeyCount) {
			CBlk(Index) = Node->Ptrs[0];
			CKey(Index) = 1;
		}
	}
/*     
printf("I%d K%ld P%ld\n", Index, *Key, *Ptr);
*/  
	}
}

long Lookup(int Index, long Key) {
	NodeType Node;
	long Ptr;
	CBlk(Index) = Root(Index);
	for(;;) {
		ReadBlock(File(Index), CBlk(Index), BLKSIZE, &Node);
		CKey(Index) = FindKey(&Node, Key);
		if(Node.LeafNode) break;
		CBlk(Index) = Node.Ptrs[CKey(Index)];
	}
	CPtr(Index) = 0;
	if(CKey(Index) == 0) CBlk(Index) = -1;
	CheckBucket(Index, &Node, &Key, &Ptr);
	return Ptr;
}



static int InsertKey(int Index, NodeType *Node, int KIdx, long *Key, long *Ptr) {
	long Keys[ORDER], Ptrs[ORDER+1];
	int Count, Count1, Count2, k;
	Count = Node->KeyCount + 1;
	Count1 = Count < ORDER ? Count : ORDER/2;
	Count2 = Count - Count1;
	for(k = ORDER/2; k < KIdx; k++) {
		Keys[k] = Node->Keys[k];
		Ptrs[k+1] = Node->Ptrs[k+1];
	}
	Keys[KIdx] = *Key;
	Ptrs[KIdx+1] = *Ptr;
	for(k = KIdx; k < Node->KeyCount; k++) {
		Keys[k+1] = Node->Keys[k];
		Ptrs[k+2] = Node->Ptrs[k+1];
	}
	for(k = KIdx; k < Count1; k++) {
		Node->Keys[k] = Keys[k];
		Node->Ptrs[k+1] = Ptrs[k+1];
	}
	Node->KeyCount = Count1;
	if(Count2) {
		int s, d;
		NodeType NNode;
		NNode.LeafNode = Node->LeafNode;
		Count2 -= !Node->LeafNode;
		for(s = ORDER/2 + !Node->LeafNode, d = 0; d < Count2; s++, d++) {
			NNode.Keys[d] = Keys[s];
			NNode.Ptrs[d] = Ptrs[s];
		}
		NNode.Ptrs[d] = Ptrs[s];
		NNode.KeyCount = Count2;
		*Key = Keys[ORDER/2];
		*Ptr = Alloc(Index)++;
		if(Node->LeafNode) {  /* insert in sequential linked list */
			NNode.Ptrs[0] = Node->Ptrs[0];
			Node->Ptrs[0] = *Ptr;
		}
		WriteBlock(File(Index), *Ptr, BLKSIZE, &NNode);
		WriteHead(Index);
	}
	return Count2;
}

static long NewBucket(int Index, long Ptr, long Next) {
	long BBlk;
        int j; 
	BucketType Bucket;
	Bucket.Pointers[0] = Ptr;
        Bucket.Pointers[1] = -1;  
	Bucket.NextBucket = Next;
	BBlk = Alloc(Index)++;
	WriteBlock(File(Index), BBlk, BLKSIZE, &Bucket);
	WriteHead(Index);
	return BBlk;
}

static int Error;

static int RecInsert(int Index, long Block, long *Key, long *Ptr) {
	NodeType Node;
	int KIdx, Split = 0;
	int EqualKey;
	ReadBlock(File(Index), Block, BLKSIZE, &Node);
	KIdx = FindKey(&Node, *Key);
	EqualKey = KIdx && Node.Keys[KIdx-1] == *Key;
	if(!Node.LeafNode)
		Split = RecInsert(Index, Node.Ptrs[KIdx], Key, Ptr);
	if(Split || Node.LeafNode && !EqualKey) {
		if(Node.LeafNode && Dups(Index))
			*Ptr = NewBucket(Index, *Ptr, -1);
		Split = InsertKey(Index, &Node, KIdx, Key, Ptr);
		WriteBlock(File(Index), Block, BLKSIZE, &Node);
	} else if(Node.LeafNode && Dups(Index)) { /* put in existing bucket */
		BucketType Bucket;
		int i;
		ReadBlock(File(Index), Node.Ptrs[KIdx], BLKSIZE, &Bucket);
		for(i = 0; i < BKTPTRS && Bucket.Pointers[i] >= 0; i++);
		if(i < BKTPTRS) {
		    Bucket.Pointers[i] = *Ptr;
		    if(i < BKTPTRS-1) Bucket.Pointers[i+1] = -1;
		    WriteBlock(File(Index), Node.Ptrs[KIdx], BLKSIZE, &Bucket);
		} else {
			printf("ERROR: Bucket Overflow\n");
		}
	} else if(Node.LeafNode) {
		Error = -1;
	}
	return Split;
}

int Insert(int Index, long Key, long Ptr) {
	int Split;
	Error = 0;
	Split = RecInsert(Index, Root(Index), &Key, &Ptr);
	if(Split) {
		NodeType Node;
		Node.LeafNode = 0;
		Node.KeyCount = 1;
		Node.Keys[0] = Key;
		Node.Ptrs[1] = Ptr;
		Node.Ptrs[0] = Root(Index);
		Root(Index) = Alloc(Index)++;
		WriteBlock(File(Index), Root(Index), BLKSIZE, &Node);
		WriteHead(Index);
	}
	CBlk(Index) = -1;
	return Error;
}

main()  {

/********************************************************************/
/* The main section of code should be replaced.                     */ 
/* It just illustrates how keys and addresses can be inserted       */   
/* into a B+ tree and then retrieved                                */   
/********************************************************************/

 char Xname[8];
  int  Xdupkeys; 
  int  XEntryNum; 
  long XKey, XPtr;    
  int  RetnCode;     
  int  i,j;     

  int NumInsert, NumLookup;  

  printf("start main\n");  

  Xdupkeys  = 0;  
  Xname[0]  = 'm'; 
  Xname[1]  = 'y'; 
  Xname[2]  = 'f'; 
  Xname[3]  = 'i'; 
  Xname[4]  = 'l'; 
  Xname[5]  = 'e'; 
  

/********************************************************************/  
/* read number of keys to insert, the number of keys  to search for */ 
/* and whether duplicates are allowed in the index                  */ 
/********************************************************************/  

  scanf("%d  %d  %d",&NumInsert, &NumLookup, &Xdupkeys);

  XEntryNum = OpenIndex(Xname, Xdupkeys);  

  for(i = 0; i < NumInsert; i++){  
		printf("Insert KEY, PTR:");
/* read key and address  to insert into the B+ tree  */ 

       scanf("%d  %d",&XKey, &XPtr);
       printf("%d  %d",XKey, XPtr);
       printf("\n");

       RetnCode = Insert(XEntryNum, XKey, XPtr);        
       if (RetnCode != 0)  
          printf("Error During Key Insertion\n");         
     }  


 for(i = 0; i < NumLookup; i++)
     {  

/* read key to lookup in the B+ tree  */ 
		printf("Search KEY:");
       scanf("%d",&XKey);
       printf("%d",XKey);
       printf("\n");
       XPtr = Lookup(XEntryNum, XKey);   
       if (Xdupkeys == 0)  {  
         printf("Key value is %d and Ptr value is %d\n", XKey, XPtr);    
        } 
       else 
         printf("Key value is %d\n", XKey);  

       if (Xdupkeys == 1) { 
         j = 0;  
         while (Bucket.Pointers[j] != -1)  { 
            printf("Ptr value is %d\n", Bucket.Pointers[j]); 
            j++; 
           } 
       }    
     }  


  CloseIndex(XEntryNum);

  printf("finish main\n");  


}

/* Copyright (C) 2006 David Garcia
 * You may use, copy, modify, merge, publish, distribute, sublicense,
 * and/or sell copies of this software subject to the following conditions:
 * THIS SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
 * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
 * IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
 * CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
 * TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE
 * OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 */

#if !defined BPLUSTREE_HPP_227824
#define BPLUSTREE_HPP_227824

// This is required for glibc to define std::posix_memalign
#if !defined(_XOPEN_SOURCE) || (_XOPEN_SOURCE < 600)
#define _XOPEN_SOURCE 600
#endif
#include <stdlib.h>

#include <assert.h>
#include <boost/static_assert.hpp>
#include <boost/pool/object_pool.hpp>

// DEBUG
#include <iostream>
using std::cout;
using std::endl;

#if defined DEBUG_ASSERT
#    warning "DEBUG_ASSERT overloaded"
#else
#    if defined DEBUG
#        define DEBUG_ASSERT(expr) assert(expr)
#    else
#        define DEBUG_ASSERT(expr)
#    endif
#endif

template <typename KEY, typename VALUE, unsigned N, unsigned M,
	unsigned INNER_NODE_PADDING= 0, unsigned LEAF_NODE_PADDING= 0,
	unsigned NODE_ALIGNMENT= 64>
class BPlusTree
{
public:
	// N must be greater than two to make the split of
	// two inner nodes sensible.
	BOOST_STATIC_ASSERT(N>2);
	// Leaf nodes must be able to hold at least one element
	BOOST_STATIC_ASSERT(M>0);

	// Builds a new empty tree.
	BPlusTree()
	: depth(0),
	  root(new_leaf_node())
	{
		// DEBUG
		// cout << "sizeof(LeafNode)==" << sizeof(LeafNode) << endl;
		// cout << "sizeof(InnerNode)==" << sizeof(InnerNode) << endl;
	}

	~BPlusTree() {
		// Empty. Memory deallocation is done automatically
		// when innerPool and leafPool are destroyed.
	}

	// Inserts a pair (key, value). If there is a previous pair with
	// the same key, the old value is overwritten with the new one.
	void insert(KEY key, VALUE value) {
		InsertionResult result;
		bool was_split;
		if( depth == 0 ) {
			// The root is a leaf node
			DEBUG_ASSERT( *reinterpret_cast<NodeType*>(root) == 
				NODE_LEAF);
			was_split= leaf_insert(reinterpret_cast<LeafNode*>
				(root), key, value, &result);
		} else {
			// The root is an inner node
			DEBUG_ASSERT( *reinterpret_cast<NodeType*>
				(root) == NODE_INNER );
			was_split= inner_insert(reinterpret_cast<InnerNode*>
				(root), depth, key, value, &result);
		}
		if( was_split ) {
			// The old root was splitted in two parts.
			// We have to create a new root pointing to them
			depth++;
			root= new_inner_node();
			InnerNode* rootProxy=
				reinterpret_cast<InnerNode*>(root);
			rootProxy->num_keys= 1;
			rootProxy->keys[0]= result.key;
			rootProxy->children[0]= result.left;
			rootProxy->children[1]= result.right;
		}
	}

	// Looks for the given key. If it is not found, it returns false,
	// if it is found, it returns true and copies the associated value
	// unless the pointer is null.
	bool find(const KEY& key, VALUE* value= 0) const {
		InnerNode* inner;
		register void* node= root;
		register unsigned d= depth, index;
		while( d-- != 0 ) {
			inner= reinterpret_cast<InnerNode*>(node);
			DEBUG_ASSERT( inner->type == NODE_INNER );
			index= inner_position_for(key, inner->keys,
				inner->num_keys);
			node= inner->children[index];
		}
		LeafNode* leaf= reinterpret_cast<LeafNode*>(node);
		DEBUG_ASSERT( leaf->type == NODE_LEAF );
		index= leaf_position_for(key, leaf->keys, leaf->num_keys);
		if( leaf->keys[index] == key ) {
			if( value != 0 ) {
				*value= leaf->values[index];
			}
			return true;
		} else {
			return false;
		}
	}

	// Returns the size of an inner node
	// It is useful when optimizing performance with cache alignment.
	unsigned sizeof_inner_node() const {
		return sizeof(InnerNode);
	}

	// Returns the size of a leaf node.
	// It is useful when optimizing performance with cache alignment.
	unsigned sizeof_leaf_node() const {
		return sizeof(LeafNode);
	}
	

private:
	// Used when debugging
	enum NodeType {NODE_INNER=0xDEADBEEF, NODE_LEAF=0xC0FFEE};

	// Leaf nodes store pairs of keys and values.
	struct LeafNode {
#ifdef DEBUG
		LeafNode() : type(NODE_LEAF), num_keys(0) {}
		const NodeType type;
#else
		LeafNode() : num_keys(0) {}
#endif
		unsigned num_keys;
		KEY      keys[M];
		VALUE    values[M];
		unsigned char _pad[LEAF_NODE_PADDING];
	};

	// Inner nodes store pointers to other nodes interleaved with keys.
	struct InnerNode {
#ifdef DEBUG
		InnerNode() : type(NODE_INNER), num_keys(0) {}
		const NodeType type;
#else
		InnerNode() : num_keys(0) {}
#endif
		unsigned num_keys;
		KEY      keys[N];
		void*    children[N+1];
		unsigned char _pad[INNER_NODE_PADDING];
	};

	// Custom allocator that returns aligned blocks of memory
	template <unsigned ALIGNMENT>
	struct AlignedMemoryAllocator {
		typedef std::size_t size_type;
		typedef std::ptrdiff_t difference_type;
		
		static char* malloc(const size_type bytes)
		{
			void* result;
			if( posix_memalign(&result, ALIGNMENT, bytes) != 0 ) {
				result= 0;
			}
			// Alternative: result= std::malloc(bytes);
			return reinterpret_cast<char*>(result);
		}
		static void free(char* const block)
		{ std::free(block); }
	};

	// Returns a pointer to a fresh leaf node.
	LeafNode* new_leaf_node() {
		LeafNode* result;
		//result= new LeafNode();
		result= leafPool.construct();
		//cout << "New LeafNode at " << result << endl;
		return result;
	}

	// Frees a leaf node previously allocated with new_leaf_node()
	void delete_leaf_node(LeafNode* node) {
		DEBUG_ASSERT( node->type == NODE_LEAF );
		//cout << "Deleting LeafNode at " << node << endl;
		// Alternatively: delete node;
		leafPool.destroy(node);
	}

	// Returns a pointer to a fresh inner node.
	InnerNode* new_inner_node() {
		InnerNode* result;
		// Alternatively: result= new InnerNode();
		result= innerPool.construct();
		//cout << "New InnerNode at " << result << endl;
		return result;
	}

	// Frees an inner node previously allocated with new_inner_node()
	void delete_inner_node(InnerNode* node) {
		DEBUG_ASSERT( node->type == NODE_INNER );
		//cout << "Deleting InnerNode at " << node << endl;
		// Alternatively: delete node;
		innerPool.destroy(node);
	}
	
	// Data type returned by the private insertion methods.
	struct InsertionResult {
		KEY key;
		void* left;
		void* right;
	};

	// Returns the position where 'key' should be inserted in a leaf node
	// that has the given keys.
	static unsigned leaf_position_for(const KEY& key, const KEY* keys,
			unsigned num_keys) {
		// Simple linear search. Faster for small values of N or M
		unsigned k= 0;
		while((k < num_keys) && (keys[k]<key)) {
			++k;
		}
		return k;
		/*
		// Binary search. It is faster when N or M is > 100,
		// but the cost of the division renders it useless
		// for smaller values of N or M.
		XXX--- needs to be re-checked because the linear search
		has changed since the last update to the following ---XXX
		int left= -1, right= num_keys, middle;
		while( right -left > 1 ) {
			middle= (left+right)/2;
			if( keys[middle] < key) {
				left= middle;
			} else {
				right= middle;
			}
		}
		//assert( right == k );
		return unsigned(right);
		*/
	}

	// Returns the position where 'key' should be inserted in an inner node
	// that has the given keys.
	static unsigned inner_position_for(const KEY& key, const KEY* keys,
			unsigned num_keys) {
		// Simple linear search. Faster for small values of N or M
		unsigned k= 0;
		while((k < num_keys) && ((keys[k]<key) || (keys[k]==key))) {
			++k;
		}
		return k;
		// Binary search is faster when N or M is > 100,
		// but the cost of the division renders it useless
		// for smaller values of N or M.
	}

	bool leaf_insert(LeafNode* node, KEY& key,
			VALUE& value, InsertionResult* result) {
		DEBUG_ASSERT( node->type == NODE_LEAF );
		assert( node->num_keys <= M );
		bool was_split= false;
		// Simple linear search
		unsigned i= leaf_position_for(key, node->keys, node->num_keys);
		if( node->num_keys == M ) {
			// The node was full. We must split it
			unsigned treshold= (M+1)/2;
			LeafNode* new_sibling= new_leaf_node();
			new_sibling->num_keys= node->num_keys -treshold;
			for(unsigned j=0; j < new_sibling->num_keys; ++j) {
				new_sibling->keys[j]= node->keys[treshold+j];
				new_sibling->values[j]=
					node->values[treshold+j];
			}
			node->num_keys= treshold;
			if( i < treshold ) {
				// Inserted element goes to left sibling
				leaf_insert_nonfull(node, key, value, i);
			} else {
				// Inserted element goes to right sibling
				leaf_insert_nonfull(new_sibling, key, value,
					i-treshold);
			}
			// Notify the parent about the split
			was_split= true;
			result->key= new_sibling->keys[0];
			result->left= node;
			result->right= new_sibling;
		} else {
			// The node was not full
			leaf_insert_nonfull(node, key, value, i);
		}
		return was_split;
	}

	static void leaf_insert_nonfull(LeafNode* node, KEY& key, VALUE& value,
			unsigned index) {
		DEBUG_ASSERT( node->type == NODE_LEAF );
		assert( node->num_keys < M );
		assert( index <= M );
		assert( index <= node->num_keys );
		if( (index < M) && (node->num_keys!=0) && (node->keys[index] == key) ) {
			// We are inserting a duplicate value.
			// Simply overwrite the old one
			node->values[index]= value;
		} else {
			// The key we are inserting is unique
			for(unsigned i=node->num_keys; i > index; --i) {
				node->keys[i]= node->keys[i-1];
				node->values[i]= node->values[i-1];
			}
			node->num_keys++;
			node->keys[index]= key;
			node->values[index]= value;
		}
	}

	bool inner_insert(InnerNode* node, unsigned depth, KEY& key,
			VALUE& value, InsertionResult* result) {
		DEBUG_ASSERT( node->type == NODE_INNER );
		assert( depth != 0 );
		// Early split if node is full.
		// This is not the canonical algorithm for B+ trees,
		// but it is simpler and does not break the definition.
		bool was_split= false;
		if( node->num_keys == N ) {
			// Split
			unsigned treshold= (N+1)/2;
			InnerNode* new_sibling= new_inner_node();
			new_sibling->num_keys= node->num_keys -treshold;
			for(unsigned i=0; i < new_sibling->num_keys; ++i) {
				new_sibling->keys[i]= node->keys[treshold+i];
				new_sibling->children[i]=
					node->children[treshold+i];
			}
			new_sibling->children[new_sibling->num_keys]=
				node->children[node->num_keys];
			node->num_keys= treshold-1;
			// Set up the return variable
			was_split= true;
			result->key= node->keys[treshold-1];
			result->left= node;
			result->right= new_sibling;
			// Now insert in the appropriate sibling
			if( key < result->key ) {
				inner_insert_nonfull(node, depth, key, value);
			} else {
				inner_insert_nonfull(new_sibling, depth, key,
					value);
			}
		} else {
			// No split
			inner_insert_nonfull(node, depth, key, value);
		}
		return was_split;
	}

	void inner_insert_nonfull(InnerNode* node, unsigned depth, KEY& key,
			VALUE& value) {
		DEBUG_ASSERT( node->type == NODE_INNER );
		assert( node->num_keys < N );
		assert( depth != 0 );
		// Simple linear search
		unsigned index= inner_position_for(key, node->keys,
			node->num_keys);
		InsertionResult result;
		bool was_split;
		if( depth-1 == 0 ) {
			// The children are leaf nodes
			for(unsigned kk=0; kk < node->num_keys+1; ++kk) {
				DEBUG_ASSERT( *reinterpret_cast<NodeType*>
					(node->children[kk]) == NODE_LEAF );
			}
			was_split= leaf_insert(reinterpret_cast<LeafNode*>
				(node->children[index]), key, value, &result);
		} else {
			// The children are inner nodes
			for(unsigned kk=0; kk < node->num_keys+1; ++kk) {
				DEBUG_ASSERT( *reinterpret_cast<NodeType*>
					(node->children[kk]) == NODE_INNER );
			}
			InnerNode* child= reinterpret_cast<InnerNode*>
				(node->children[index]);
			was_split= inner_insert( child, depth-1, key, value,
				&result);
		}
		if( was_split ) {
			if( index == node->num_keys ) {
				// Insertion at the rightmost key
				node->keys[index]= result.key;
				node->children[index]= result.left;
				node->children[index+1]= result.right;
				node->num_keys++;
			} else {
				// Insertion not at the rightmost key
				node->children[node->num_keys+1]=
					node->children[node->num_keys];
				for(unsigned i=node->num_keys; i!=index; --i) {
					node->children[i]= node->children[i-1];
					node->keys[i]= node->keys[i-1];
				}
				node->children[index]= result.left;
				node->children[index+1]= result.right;
				node->keys[index]= result.key;
				node->num_keys++;
			}
		} // else the current node is not affected
	}

	typedef AlignedMemoryAllocator<NODE_ALIGNMENT> AlignedAllocator;

	// Node memory allocators. IMPORTANT NOTE: they must be declared
	// before the root to make sure that they are properly initialised
	// before being used to allocate any node.
	boost::object_pool<InnerNode, AlignedAllocator> innerPool;
	boost::object_pool<LeafNode, AlignedAllocator>  leafPool;
	// Depth of the tree. A tree of depth 0 only has a leaf node.
	unsigned depth;
	// Pointer to the root node. It may be a leaf or an inner node, but
	// it is never null.
	void*    root;
};

#endif // !defined BPLUSTREE_HPP_227824
