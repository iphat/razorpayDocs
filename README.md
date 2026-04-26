# razorpayDocs
 # Whole code is wiped out from a website(snitch)  code so may be some part is according to the website requirement

BACKEND 

step1 -
         
    npm i razorpay

step2-
prototype credentials

    RAZORPAY_KEY_ID="razorpay id "
    RAZORPAY_KEY_SECRET="razorpay secret"
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

  step6 - 
  API - if we are doing payment for cart items 
  
     import { authenticateUser } from '../middlewares/auth.middleware.js';
     import { createOrderController} from '../controllers/cart.controller.js';
     
     const router = express.Router();
     router.post("/payment/create/order", authenticateUser, createOrderController)





 FRONTEND -
 step0-
 src > features > cart > state > cart.slice.js
 create a reducer first 

 step1-
 Frontend > src > cart > state > cart.api.js
 
     import axios from "axios";

     const cartApiInstance = axios.create({
         baseURL: "/api/cart",
         withCredentials: true
      })
       
      export const createCartOrder = async()=>{
        const response = await cartApiInstance.post("/payment/create/order")
         return response.data
       }


step2-

    import  {createCartOrder } from "../service/cart.api.js"
    import {useDispatch} from  "react-redux"

    export const useCart = () => {
    const dispatch = useDispatch()

    
    async function handleCreateCartOrder(){
        const data = await createCartOrder()
        return data.order
    }   

    return {handleAddItem, handleGetCart, handleIncrementCartItemApi, handleCreateCartOrder }
    }


step3-

    npm i react-razorpay

step4- 
Frontend > features > cart > pages > Cart.jsx

    import { useCart } from '../hook/useCart'
    import { Link, useNavigate } from 'react-router'
    import { useRazorpay, RazorpayOrderOptions } from "react-razorpay";

    const Cart = () => {
    const cart = useSelector(state => state.cart)
    const { handleCreateCartOrder } = useCart()
    const navigate = useNavigate()
    // this hook is used to load the Razorpay script and manage the loading state and errors.
    const { error, isLoading, Razorpay } = useRazorpay();
    const user = useSelector(state => state.user)
    
        // razorpay integration
        
        // razorpay integration
    async function handleCheckout() {
        try {
            const order = await handleCreateCartOrder()
            if (!order) {
                console.error("Failed to create order")
                return
            }
            console.log("order is here", order)

            if (!Razorpay) {
                console.error("Razorpay is not loaded yet")
                return
            }

            // react-razorpay 
            const options = {
                key: "rzp_test_Si7S8lnLlFIivK",
                amount: order.amount, // Amount in paise
                currency: order.currency,
                name: "Snitch",
                description: "Test Transaction",
                order_id: order.id, // Generate order_id on server
                handler: (response) => {
                    console.log(response);
                    alert("Payment Successful!");
                },
                prefill: {
                    name: user?.fullname,
                    email: user?.email,
                    contact: user?.contact,
                },
                theme: {
                    color: tokens.primary,
                },
            };

            const razorpayInstance = new Razorpay(options);
            razorpayInstance.open();
        } catch (err) {
            console.error("Checkout error:", err)
        }
    }

       return (
        <>
        <button  onClick={handleCheckout}>  Proceed to Checkout </button>
        </>
        )
