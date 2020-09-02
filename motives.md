# Pipeline Implementation Approaches


# Object-Oriented Programming 

Pipeline is expressed through objects and their methods (there should not be functions not coupled with some object)

## Cons
- The rest of the app is not pure object oriented
- OOP is too verbose and implicit. The process of execution is not evident.
## Current implementation violates
- ZEN_1. Explicit is better than implicit.
- ZEN_17. If the implementation is hard to explain, it's a bad idea.
- SOLID_1. Single responsibility principle (Basic programming language entity [function, class, or object] should have one and only one reason to change, meaning that a it should have only one job being performed.

## Because
1. [No single responsibility] Using a single entity (Transform) to solve the thre separate problems: making the job sequence (1) distributed, (2) chainable and (3) laze.
2. [No freedom and flexibility] Your terminology makes me believe that everything I want to be distributed will be lazily chained. What if the user wants to chain the tasks without distributing or to distribute without chaining?


## Adapter Pattern Approach
```
from hub import dataset, Transform

class Transform:
    def __init__(self):
        pass

    def meta(self):
        raise NotImplementedError()

    def call(input): #execute, handle, perform, compute
        raise NotImplementedError()


class Pipeline:
    def __init__(self):
        self.transforms = []

    def add(self, transforms):
        self.transforms += [transform]

    def forward(input):
        # apply transforms sequentially


# Main Difference from PyTorch Case: long class hierarchies are needed to express the variativity of pipelines. 
# DelayedDistributedPipeline -> DistributedPipeline -> Pipeline


class Crop(Transform):

   def meta(self):
      return {"image": {"shape": (1, 256, 256), "dtype": "uint8"}}

   def call(self, input):
      return {"image": input[0:1, :256, :256]}



class Flip(Transform):

   def call(self, input):
      img = np.expand_dims(input["image"], axis=0)
      img = np.flip(img, axis=(1, 2))
      return {"image": img}

   def meta(self):
      return {"image": {"shape": (1, 256, 256), "dtype": "uint8"}}


from hub import 

pipeline = Pipieline()
pipeline.add(Crop())
pipeline.add(Flip())
ds = dataset.generate(Crop(), images)
ds = dataset.generate(Flip(), ds)
ds.store("/tmp/cropflip")
```

# Builder Pattern
Constructs the object sequentually by applying methods.
```
import hub

ds = hub
    .load('./tmp')
    .with(meta)     # in meta info is not specified you could try to do it by your own
    .apply(crop)    # staking first function
    .apply(flip)    # stacking second function
    .distribute()   # for distributing with dask
    .chain()        # finish the build and chain functions for delayed evaluation 

ds.compute()        #
```


# Functional programming

## Decorator Pattern
```
@delayed
@distributed('dask', n_workers=5)
def crop(input):
    pass

@delayed
@distributed('dask', meta, n_workers=5)
def flip(input):
    pass


ds = hub.load('./tmp')
ds = crop(ds)
ds = flop(ds)
ds.compute()
```

## Monad Pattern
Monad is a monoid in the category of endofunctors.
Put it in simple terms monad is a type constructor with return and pipe operators that could help to stack transforms and apply their superposition in the reversed order

```
import mPyPl as mp

def crop(input):
    pass

def flip(input):
    pass

ds = (
      mp.load('./tmp')
    | mp.with(meta)
    | mp.apply(crop)
    | mp.apply(flip)
    | mp.distribute     # for distributing with dask
    | mp.chain          # for lazy eval of the chain
)
mp.compute(ds)
```

