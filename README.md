use client'

import React, { useState, useRef } from 'react'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Card } from '@/components/ui/card'
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'

interface InvoiceItem {
  id: string
  description: string
  quantity: number
  unitPrice: number
  totalPrice: number
}

interface InvoiceData {
  companyName: string
  invoiceNumber: string
  accountNumber: string
  invoiceDate: string
  clientName: string
  clientMobile: string
  clientEmail: string
  items: InvoiceItem[]
  paymentMethod: string
  accountDetails: string
  swiftCode: string
  payoneerDetails: string
  subtotal: number
  vatRate: number
  discountRate: number
  directorName: string
  phone: string
  email: string
  website: string
  address: string
}

export default function InvoiceEditor() {
  const invoiceRef = useRef<HTMLDivElement>(null)
  
  const [invoiceData, setInvoiceData] = useState<InvoiceData>({
    companyName: 'CHARA MARVELS',
    invoiceNumber: '001',
    accountNumber: 'xxxxxx',
    invoiceDate: 'xxxxxx',
    clientName: '',
    clientMobile: '',
    clientEmail: '',
    items: [
      { id: '1', description: '', quantity: 0, unitPrice: 0, totalPrice: 0 }
    ],
    paymentMethod: '',
    accountDetails: 'xxxxxxxxxxxx',
    swiftCode: 'xxxxxxxxxxxx',
    payoneerDetails: 'xxxxxxxxxxxx',
    subtotal: 0,
    vatRate: 10,
    discountRate: 5,
    directorName: 'BUSI.RAMESH',
    phone: '+000 12345 6789',
    email: 'unwebsitename.com',
    website: 'unwebsitename.com',
    address: 'Street Address write Here.100'
  })

  const [editingField, setEditingField] = useState<string | null>(null)

  const updateField = (field: string, value: string | number) => {
    setInvoiceData(prev => ({
      ...prev,
      [field]: value
    }))
  }

  const updateItem = (id: string, field: keyof InvoiceItem, value: string | number) => {
    setInvoiceData(prev => ({
      ...prev,
      items: prev.items.map(item => {
        if (item.id === id) {
          const updatedItem = { ...item, [field]: value }
          if (field === 'quantity' || field === 'unitPrice') {
            updatedItem.totalPrice = updatedItem.quantity * updatedItem.unitPrice
          }
          return updatedItem
        }
        return item
      })
    }))
  }

  const addItem = () => {
    const newItem: InvoiceItem = {
      id: Date.now().toString(),
      description: '',
      quantity: 0,
      unitPrice: 0,
      totalPrice: 0
    }
    setInvoiceData(prev => ({
      ...prev,
      items: [...prev.items, newItem]
    }))
  }

  const removeItem = (id: string) => {
    setInvoiceData(prev => ({
      ...prev,
      items: prev.items.filter(item => item.id !== id)
    }))
  }

  const calculateSubtotal = () => {
    return invoiceData.items.reduce((sum, item) => sum + item.totalPrice, 0)
  }

  const calculateVat = () => {
    return (calculateSubtotal() * invoiceData.vatRate) / 100
  }

  const calculateDiscount = () => {
    return (calculateSubtotal() * invoiceData.discountRate) / 100
  }

  const calculateGrandTotal = () => {
    return calculateSubtotal() + calculateVat() - calculateDiscount()
  }

  const downloadPDF = async () => {
    if (!invoiceRef.current) return

    try {
      const canvas = await html2canvas(invoiceRef.current, {
        scale: 2,
        useCORS: true,
        allowTaint: true
      })
      
      const imgData = canvas.toDataURL('image/png')
      const pdf = new jsPDF('p', 'mm', 'a4')
      const imgWidth = 210
      const pageHeight = 295
      const imgHeight = (canvas.height * imgWidth) / canvas.width
      let heightLeft = imgHeight

      let position = 0

      pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight)
      heightLeft -= pageHeight

      while (heightLeft >= 0) {
        position = heightLeft - imgHeight
        pdf.addPage()
        pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight)
        heightLeft -= pageHeight
      }

      pdf.save(`invoice-${invoiceData.invoiceNumber}.pdf`)
    } catch (error) {
      console.error('Error generating PDF:', error)
    }
  }

  const EditableField = ({ 
    value, 
    field, 
    className = "", 
    multiline = false,
    type = "text"
  }: { 
    value: string | number
    field: string
    className?: string
    multiline?: boolean
    type?: string
  }) => {
    const isEditing = editingField === field

    if (isEditing) {
      return multiline ? (
        <Textarea
          value={value}
          onChange={(e) => updateField(field, e.target.value)}
          onBlur={() => setEditingField(null)}
          onKeyDown={(e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
              setEditingField(null)
            }
          }}
          className={`${className} min-h-[40px]`}
          autoFocus
        />
      ) : (
        <Input
          type={type}
          value={value}
          onChange={(e) => updateField(field, type === 'number' ? parseFloat(e.target.value) || 0 : e.target.value)}
          onBlur={() => setEditingField(null)}
          onKeyDown={(e) => {
            if (e.key === 'Enter') {
              setEditingField(null)
            }
          }}
          className={className}
          autoFocus
        />
      )
    }

    return (
      <div
        onClick={() => setEditingField(field)}
        className={`${className} cursor-pointer hover:bg-gray-100 rounded px-2 py-1 min-h-[40px] flex items-center`}
      >
        {value || 'Click to edit'}
      </div>
    )
  }

  return (
    <div className="min-h-screen bg-gray-50 p-4">
      <div className="max-w-4xl mx-auto">
        <div className="mb-6 flex justify-between items-center">
          <h1 className="text-3xl font-bold text-gray-800">Invoice Editor</h1>
          <Button onClick={downloadPDF} className="bg-blue-600 hover:bg-blue-700">
            Download PDF
          </Button>
        </div>

        <Card className="p-0 overflow-hidden">
          <div ref={invoiceRef} className="bg-white">
            {/* Header */}
            <div className="relative bg-gradient-to-r from-blue-600 to-blue-500 text-white p-8">
              <div className="absolute top-0 right-0 w-32 h-32 bg-yellow-400 transform rotate-45 translate-x-16 -translate-y-16"></div>
              <div className="flex justify-between items-start relative z-10">
                <div>
                  <div className="w-16 h-16 bg-white rounded-full flex items-center justify-center mb-4">
                    <div className="text-blue-600 font-bold text-xl">CM</div>
                  </div>
                  <EditableField
                    value={invoiceData.companyName}
                    field="companyName"
                    className="text-sm text-blue-100"
                  />
                </div>
                <div className="text-right">
                  <h1 className="text-4xl font-bold mb-2">Invoice</h1>
                </div>
              </div>
            </div>

            {/* Invoice Details */}
            <div className="p-8">
              <div className="flex justify-between mb-8">
                <div>
                  <h2 className="text-lg font-semibold mb-4">Invoice To:</h2>
                  <div className="space-y-2">
                    <EditableField
                      value={invoiceData.clientName}
                      field="clientName"
                      className="font-medium"
                    />
                    <div>
                      <span className="text-sm text-gray-600">Mobile No: </span>
                      <EditableField
                        value={invoiceData.clientMobile}
                        field="clientMobile"
                        className="inline-block"
                      />
                    </div>
                    <div>
                      <span className="text-sm text-gray-600">Email ID: </span>
                      <EditableField
                        value={invoiceData.clientEmail}
                        field="clientEmail"
                        className="inline-block"
                      />
                    </div>
                  </div>
                </div>
                <div className="text-right">
                  <div className="bg-gray-800 text-white px-4 py-2 rounded mb-2">
                    <span className="text-sm">Invoice No: </span>
                    <EditableField
                      value={invoiceData.invoiceNumber}
                      field="invoiceNumber"
                      className="inline-block text-white bg-transparent border-none"
                    />
                  </div>
                  <div className="space-y-1 text-sm">
                    <div>
                      <span className="text-gray-600">Account No: </span>
                      <EditableField
                        value={invoiceData.accountNumber}
                        field="accountNumber"
                        className="inline-block"
                      />
                    </div>
                    <div>
                      <span className="text-gray-600">Invoice Date: </span>
                      <EditableField
                        value={invoiceData.invoiceDate}
                        field="invoiceDate"
                        className="inline-block"
                        type="date"
                      />
                    </div>
                  </div>
                </div>
              </div>

              {/* Items Table */}
              <div className="mb-8">
                <table className="w-full border-collapse">
                  <thead>
                    <tr className="bg-blue-600 text-white">
                      <th className="border border-gray-300 p-3 text-left">Item description</th>
                      <th className="border border-gray-300 p-3 text-center">Quantity</th>
                      <th className="border border-gray-300 p-3 text-center">Unit Price</th>
                      <th className="border border-gray-300 p-3 text-center">Total Price</th>
                      <th className="border border-gray-300 p-3 text-center">Actions</th>
                    </tr>
                  </thead>
                  <tbody>
                    {invoiceData.items.map((item) => (
                      <tr key={item.id} className="border-b">
                        <td className="border border-gray-300 p-3">
                          <Input
                            value={item.description}
                            onChange={(e) => updateItem(item.id, 'description', e.target.value)}
                            placeholder="Enter item description"
                            className="border-none bg-transparent"
                          />
                        </td>
                        <td className="border border-gray-300 p-3 text-center">
                          <Input
                            type="number"
                            value={item.quantity}
                            onChange={(e) => updateItem(item.id, 'quantity', parseFloat(e.target.value) || 0)}
                            className="border-none bg-transparent text-center"
                          />
                        </td>
                        <td className="border border-gray-300 p-3 text-center">
                          <Input
                            type="number"
                            step="0.01"
                            value={item.unitPrice}
                            onChange={(e) => updateItem(item.id, 'unitPrice', parseFloat(e.target.value) || 0)}
                            className="border-none bg-transparent text-center"
                          />
                        </td>
                        <td className="border border-gray-300 p-3 text-center font-medium">
                          ${item.totalPrice.toFixed(2)}
                        </td>
                        <td className="border border-gray-300 p-3 text-center">
                          <Button
                            onClick={() => removeItem(item.id)}
                            variant="destructive"
                            size="sm"
                            disabled={invoiceData.items.length === 1}
                          >
                            Remove
                          </Button>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
                <Button onClick={addItem} className="mt-4" variant="outline">
                  Add Item
                </Button>
              </div>

              {/* Payment Details and Totals */}
              <div className="flex justify-between">
                <div className="w-1/2">
                  <h3 className="font-semibold mb-2">Payment method</h3>
                  <div className="space-y-2 text-sm">
                    <div>
                      <span className="text-gray-600">Account: </span>
                      <EditableField
                        value={invoiceData.accountDetails}
                        field="accountDetails"
                        className="inline-block"
                      />
                    </div>
                    <div>
                      <span className="text-gray-600">Swift: </span>
                      <EditableField
                        value={invoiceData.swiftCode}
                        field="swiftCode"
                        className="inline-block"
                      />
                    </div>
                    <div>
                      <span className="text-gray-600">Payoneer: </span>
                      <EditableField
                        value={invoiceData.payoneerDetails}
                        field="payoneerDetails"
                        className="inline-block"
                      />
                    </div>
                  </div>
                </div>
                <div className="w-1/3">
                  <div className="space-y-2 text-sm">
                    <div className="flex justify-between">
                      <span>Sub Total</span>
                      <span>${calculateSubtotal().toFixed(2)}</span>
                    </div>
                    <div className="flex justify-between">
                      <span>Vat & Tax {invoiceData.vatRate}%</span>
                      <span>${calculateVat().toFixed(2)}</span>
                    </div>
                    <div className="flex justify-between">
                      <span>Discount {invoiceData.discountRate}%</span>
                      <span>-${calculateDiscount().toFixed(2)}</span>
                    </div>
                  </div>
                  <div className="bg-blue-600 text-white p-3 rounded mt-4">
                    <div className="flex justify-between font-bold">
                      <span>Grand Total</span>
                      <span>${calculateGrandTotal().toFixed(2)}</span>
                    </div>
                  </div>
                </div>
              </div>

              {/* Footer */}
              <div className="mt-12 pt-8 border-t">
                <div className="text-center mb-6">
                  <EditableField
                    value={invoiceData.directorName}
                    field="directorName"
                    className="font-bold text-lg inline-block"
                  />
                  <div className="text-sm text-gray-600">Director</div>
                </div>
                
                <div className="text-sm text-gray-600 mb-4">
                  <p><strong>Note:</strong> Drug once sold will not be taken back or exchanged</p>
                  <p>Thanks for your business!</p>
                </div>

                <div className="flex justify-center space-x-8 text-sm">
                  <div className="flex items-center">
                    <span className="w-4 h-4 bg-yellow-400 rounded-full mr-2"></span>
                    <EditableField
                      value={invoiceData.phone}
                      field="phone"
                      className="inline-block"
                    />
                  </div>
                  <div className="flex items-center">
                    <span className="w-4 h-4 bg-yellow-400 rounded-full mr-2"></span>
                    <EditableField
                      value={invoiceData.email}
                      field="email"
                      className="inline-block"
                    />
                  </div>
                  <div className="flex items-center">
                    <span className="w-4 h-4 bg-yellow-400 rounded-full mr-2"></span>
                    <EditableField
                      value={invoiceData.address}
                      field="address"
                      className="inline-block"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </Card>
      </div>
    </div>
  )
}
