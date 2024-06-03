
# How does the model learn, is the question .🤔🤔

This article will give you a small intuition on how the model learn .

step1 : predict the values

step 2 : calculate the loss

step 3 : go backward on the loss . i’m not kidding .

step 4 : calculate the gradient

step 5 : optimize the parameter

What matters the most for prediction of the values is the parameter values .

    import torch
    import numpy
    import matplotlib.pyplot as plt

    time=torch.arange(0,20).float() # we have the variable time which is of 20s 
    speed=torch.randn(20) + 0.75*(time - 9.5)**2 + 1
    #so "speed" above is what we want to predict for a range of 20s as indicated by time

    def f(time,params):
              t=time
              a,b,c=params 
             return a*(t**2) + b*(t) + c
    #this is the function used to predict the value of speed for every second t

    def mse(pred,targets):
            return ((targets - pred)**2).mean()
    #this here is our loss function . loss function depends on prediction which depends on f()
    
    params = torch.randn(3).requires_grad_()
    orig_params=params.clone()

in “params = torch.randn(3).requires_grad_()” if the “requires_grad_()” isnt added , then the pytorch will not be able to find the derivate of something w.r.t params . Hence this line of code is massively important .

    def show_preds(preds,ax=None):
            if ax is None:
            ax=plt.subplots()[1]
    ax.scatter(time,speed)
    ax.scatter(time,preds.detach().numpy(),color='red')
    ax.set_ylim(-300,200)
    ax.set_xlim(0,20)
    ax.plot()
    plt.show()

![](https://cdn-images-1.medium.com/max/2000/1*9nHluUD-cZULrBaDIpsBAg.jpeg)

What does the loss function do ?

→ calculate the loss , that is indicating the difference between predicted values and real values . Our goal is to reduce this value of loss with respect to the parameter

what does loss.backward() ?

loss.backward() tells pytorch that we are going to compute the derivate of loss.

what are params here ?

params here are parameter values that model depends on to produce the output , initially they are random values that are generated but over a period of time after undergoing training where we compute the gradient , and perform optimization on parameter values , this initial random value takes on a shape of a very sophisticated value without which the model will not produce anything of value .

    lr =1e-5
    def apply_step(params,prn=True):
        preds=f(time,params)#predict the values :step 1
        print("preds",preds)
        loss=mse(speed,preds)#calculaye the loss : step 2
        print("loss ",loss)
        loss.backward()#go backward on loss : step 3
        params.grad#calculate the gradient with respect to the parameter : step 4
        print("params.grad :" ,params.grad)
        params.data -= params.grad.data*(lr) # update the parameter value towards the direction of less loss 
        params.grad=None # prams.grad=None otherwise the derivate value will keep on adding 
        print("\n\n")
        return params
        print("\n\n")

What does the loss.backward() and params.grad actually do .

loss.backward() tells pytorch that find the derivative of “loss”

loss=mse(pred,targets)

mse(pred,targets)=return ( (targets — pred )**2).mean()

pred = f(time,params)

f(time,params)= a*(t**2) + b*(t) + c

which implies loss = a*(t**2) + b*(t) + c

loss.backward() — -> derivative of (a*(t**2) + b*(t) + c) w.r.t what ???

so params .grad completes the what part of it , it calculate the derivative w.r.t params and substitutes the values for each of the parameter and finds the value .

so derivative means , rate of change of y w.r.t x , in the above case we have 2 conditions that is the derivative is -ve which means that rate of change of loss w.r.t parameter is -ve , we want to go towards this side of the parameter value which reduces the loss, so we subtract the -ve value from params original value , where we end up adding the derivative value and moving the parameter value in the direction of reduced loss , that is toward the right +ve x axis

    params.data =params.data -  (- params.grad.data*(lr)) 

case 2 :that is the derivative is +ve which means that rate of change of loss w.r.t parameter is +ve , we want to go towards this side of the parameter value which reduces the loss, so we subtract the +ve value from params original value , where we end up adding the derivative value and moving the parameter value in the direction of reduced loss . that is toward the right -ve x axis

    params.data =params.data -  ( params.grad.data*(lr))

    for epoch in range(200):
    params=apply_step(params)
    print(speed)

please reach out to adithyabhatl1998@gmail.com for any clarifications
