# Connected Component Analysis

### 1-pass Algorithm 

```
One-Pass(Image)
            [M, N]=size(Image);
            Connected = zeros(M,N);
            Mark = Value;
            Difference = Increment;
            Offsets = [-1; M; 1; -M];
            Index = [];
            No_of_Objects = 0;

   for i: 1:M :
       for j: 1:N:
            if(Image(i,j)==1)
                 No_of_Objects = No_of_Objects +1;
                 Index = [((j-1)*M + i)];
                 Connected(Index)=Mark;
                 while ~isempty(Index)
                      Image(Index)=0;
                      Neighbors = bsxfun(@plus, Index, Offsets');
                      Neighbors = unique(Neighbors(:));
                      Index = Neighbors(find(Image(Neighbors)));
                      Connected(Index)=Mark;
                 end
                 Mark = Mark + Difference;
            end
      end
  end
```

### 2-passes Algorithm for CCL (Connected Component Labelling):
```
algorithm TwoPass(data)
   linked = []
   labels = structure with dimensions of data, initialized with the value of Background

   First pass

   for row in data:
       for column in row:
           if data[row][column] is not Background

               neighbors = connected elements with the current element's value

               if neighbors is empty
                   linked[NextLabel] = set containing NextLabel
                   labels[row][column] = NextLabel
                   NextLabel += 1

               else

                   Find the smallest label

                   L = neighbors labels
                   labels[row][column] = min(L)
                   for label in L
                       linked[label] = union(linked[label], L)

   Second pass

   for row in data
       for column in row
           if data[row][column] is not Background
               labels[row][column] = find(labels[row][column])

   return labels
```
