# razorpayDocs

# razorpayDocs

BACKEND 

step1 -
npm i razorpay

step2-

RAZORPAY_KEY_ID
RAZORPAY_KEY_SECRET

copy these two credentials from razorpay

step3-
ensure that credentials are present in .env
if not throw a custom err

config.js

    import dotenv from "dotenv"
    dotenv.config()

    if(!process.env.RAZORPAY_KEY_ID){
    throw new error ("RAZORPAY_KEY_ID is not defined in your environment variables")
    }
    if(!process.env.RAZORPAY_KEY_SECRET){
    throw new error("RAZORPAY_KEY_SECRET is not defined in your environment variables")
    }


    export const config = {
       RAZORPAY_KEY_SECRET:process.env.RAZORPAY_KEY_SECRET
    }

step4-

Backend > src > service > payment.service.js

    import Razorpay from "razorpay"
    import {config} from "../config/config.js"


    const razorpay = new Razorpay({
       key_id: config.RAZORPAY_KEY_ID,
       key_secret: config.RAZORPAY_KEY_SECRET
    })


    export const createOrder = async ({amount, currecy="INR"}) => {
       const options = {
           amount: amount * 100,//amount * 100 => 1INR * 100 = 100paisa
           currency,
       }
       const order = await razorpay.orders.create(options)
      return order
    }


step5-

controller.js

    import {createOrder} from "../services/payment.service.js";

    export const createOrderController = async (req,res) =>{
        const order = await createOrder({amount : 1000, currency: "INR"})
      return res.status(200).json({
           message: "Order created successfully",
           success: true,
            order
       })
    }
