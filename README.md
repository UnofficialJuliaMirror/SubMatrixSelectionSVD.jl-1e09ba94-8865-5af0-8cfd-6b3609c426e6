# SMSSVD.jl
[SubMatrix Selection Singular Value Decomposition](http://arxiv.org/abs/1710.08144).

## Installation
```julia
Pkg.clone("https://github.com/rasmushenningsson/SMSSVD.jl.git")
```

## Example
```julia
using SMSSVD, DataFrames, Gadfly

# Create matrices with orthonormal columns
function randorthonormal(P::Integer, N::Integer)
    @assert P≥N
    O = zeros(P,N)
    for k=1:N
        x = randn(P)
        x -= O[:,1:k-1]*(O[:,1:k-1]'x)
        O[:,k] = x/norm(x)
    end
    O
end

# Create data matrix corrupted by noise
P = 1000
N = 40
d = 4
u = zeros(P,d)
u[1:100,1:2]   = randorthonormal(100,2)
u[101:200,3:4] = randorthonormal(100,2)
s = [10,8,5,4] # singular values
v = randorthonormal(N,d)
X = u*diagm(s)*v' + 0.1*randn(P,N).*rand(P) # different strength of noise for different variables

# Compute the SMSSVD of X
σThresholds = logspace(-2,0,100)
U,Σ,V,ps,signalDimensions = smssvd(X, d, σThresholds)

# Projection Score Plot
df = DataFrame(Sigma=repmat(σThresholds',d,1)[:], ProjectionScore=ps[:], NbrDims=repmat(1:d,1,length(σThresholds))[:])
coords = Coord.cartesian(xmin=log10(σThresholds[1]), xmax=log10(σThresholds[end]), ymin=0)
plot(df,x=:Sigma,y=:ProjectionScore,color=:NbrDims,Geom.line,coords,Scale.x_log10,Guide.xlabel("σ Threshold"),Guide.ylabel("Projection Score"),Guide.colorkey("Dimension"),Guide.title("Projection Score"))
```
