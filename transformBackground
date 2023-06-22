import axios from 'axios'
import sizeOf from 'image-size'
import fs from 'fs'
import FormData from 'form-data'
import { Buffer } from 'buffer'

function getImageDimensions(base64File: string) {
  const str = base64File.split(';base64').pop()
  if (!str) {
    // TODO: handle invalid image file format
    return null
  }
  const img = Buffer.from(str, 'base64')
  const dimensions = sizeOf(img)
  return dimensions
}

const findClosestHexagon = (width: any, height: any) => {
  const imageRatio = width / height
  const hexagons = [0.5625, 0.66, 0.75, 1, 1.25, 1.3, 1.4, 1.5, 1.7, 1.77, 2]
  const closest = hexagons.reduce(function (prev, curr) {
    return Math.abs(curr - imageRatio) < Math.abs(prev - imageRatio) ? curr : prev
  })
  return `hexagon_${closest}.png`
}

function downloadImageFromUrl(url: string) {
  return axios
    .get(url, {
      responseType: 'arraybuffer',
    })
    .then(response => Buffer.from(response.data, 'binary').toString('base64'))
}

export const transformImage = async (avatarUrl: string) => {
  const base64File = await downloadImageFromUrl(avatarUrl)
  const dimensions: any = await getImageDimensions(base64File)
  const hexagonString = findClosestHexagon(dimensions.w, dimensions.h)
  const formData = new FormData()
  formData.append('image_url', avatarUrl)
  formData.append('size', 'auto')
  formData.append('scale', '75%')
  formData.append('position', '50% 75%')
  formData.append('format', 'png')

  const bgImageFile = fs.createReadStream(`./images/ignition/${hexagonString}`)

  formData.append('bg_image_file', bgImageFile, 'hello')
  try {
    const removedBackground = await axios({
      url: `https://api.remove.bg/v1.0/removebg`,
      method: 'POST',
      data: formData,
      responseType: 'arraybuffer',
      headers: {
        ...formData.getHeaders(),
        'X-API-Key': process.env.REMOVE_BG_API_KEY,
      },
    })
    return Buffer.from(removedBackground.data)
  } catch (e) {
    console.log('Error', e)
    return ''
  }
}
